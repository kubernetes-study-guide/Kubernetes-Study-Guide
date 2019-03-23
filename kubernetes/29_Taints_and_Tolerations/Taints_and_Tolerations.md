# Taints and Tolerations (eg1-taints)

Earlier we saw how we can use nodeSelector, and Pod/Node Affinity to attract new pod deployments to certain worker nodes. 

We also looked at `podAntiAffinity` which is a mechanism to deploy new pod deployments away from a group of other existing pods, i.e. it repels them. Taints and Tolerations are other mechanisms for repelling pods from being deployed to certain kube nodes.


## Taints

Taint is a setting you can enable on a kube node and is used to tell Kubernetes to don't deploy anymore pods to the node.

For example, as part of provisioning a kubecluster using kubeadm, a number of Kubernetes internal pods are deployed on the master node itself, in the kube-system namespace. As soon as that's done, the kubeadmin init process taints so the kube master that no further pods get's deployed to them. Here's an example of what a taint looks like:

```bash
root@kube-master:~# kubectl describe nodes kube-master | grep Taints
Taints:             node-role.kubernetes.io/master:NoSchedule
```

You enable taint on a node using the taint subcommand, here's the syntax:

```bash
kubectl taint node nodename key=value:Taint-type
```

There are a few different types of taints:

**NoSchedule**: Don't deploy anymore pods on in future. But existing pods can stay
**PreferNoSchedule**: Soft rule equivalent of NoSchedule
**NoExecute**: Evict any existing pods from the node. This ends up deleting the pods and recreating them on other available worker nodes. 



The key-value is something you are free to choose. For example you added a new worker node (running on with special hardware) to your cluster and you want your Dev Team want to assess whether it is suitable to use, before making it widely available. In that scenario you will want to taint it with something like this:

```bash
$ kubectl taint node kube-worker1 TrialNode=SpecialNodeForDevTeam:NoSchedule

$ kubectl describe nodes kube-worker1 | grep Taints
Taints:             TrialNode=SpecialNodeForDevTeam:NoSchedule
```

Then the Dev team performs their tests, and if it passes, then you untaint the node, see `kubectl taint --help` for how to do that.

After a node has been tainted, if you try doing a standard 3-pod-deployment, you'll find:

```bash
# kubectl apply -f configs/eg1-taints/
deployment.apps/dep-httpd created
# kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
dep-httpd-78f54cc967-2r7p8   1/1     Running   0          23s   192.168.2.13   kube-worker2   <none>           <none>
dep-httpd-78f54cc967-blznk   1/1     Running   0          23s   192.168.2.12   kube-worker2   <none>           <none>
dep-httpd-78f54cc967-gwth2   1/1     Running   0          23s   192.168.2.14   kube-worker2   <none>           <none>
```

None of the pods ended up on the tainted node, kube-worker1. 


**Question**: How will the Dev Team perform test on the tainted node if the tainted node repels all pod objects?
**Answer**: You can add a 'toleration' setting in your pod yaml spec to override taint setting.  


## Toleration

Toleration is setting you can apply to our pods using `pod.spec.tolerations`. Here's an example:


```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx_webserver
  template:
    metadata:
      labels:
        app: nginx_webserver
    spec: 
      tolerations:                    # add toleration here.
        - key: TrialNode                     # All this info is used to 
          operator: Equal                    # match particular taint 
          value: SpecialNodeForDevTeam       # setting. 
          effect: NoSchedule
      containers:
        - name: cntr-nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

Notice we had to add some matching condtions. That's because you can apply multiple taints to a node. This toleration basically cancel's out one-of the node's taint if there is a match. 


```bash
kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
dep-httpd-78f54cc967-56f68   1/1     Running   0          75s   192.168.2.17   kube-worker2   <none>           <none>
dep-httpd-78f54cc967-9kxhm   1/1     Running   0          75s   192.168.2.18   kube-worker2   <none>           <none>
dep-httpd-78f54cc967-m22th   1/1     Running   0          75s   192.168.2.16   kube-worker2   <none>           <none>
dep-nginx-7579cddb84-cfn7x   1/1     Running   0          46s   192.168.1.10   kube-worker1   <none>           <none>
dep-nginx-7579cddb84-nv4xb   1/1     Running   0          46s   192.168.2.19   kube-worker2   <none>           <none>
```
If a node has multiple taints, then your pod spec needs to have multiple tolerations in order to cancel them all out. 























