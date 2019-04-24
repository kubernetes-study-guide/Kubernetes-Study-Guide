# Configuring the Kube-Scheduler


In (a minikube or) a kubeadm provisioned kube cluster, the kube-scheduler comes in the form of a pod:

```bash
root@kube-master:~# kubectl get pods --namespace=kube-system -o wide | grep scheduler
kube-scheduler-kube-master            1/1     Running   1          42h   10.2.5.110    kube-master    <none>           <none>
```

The kube-scheduler's main job is to constantly loops through a new-pods-waiting-to-be-scheduled queue, take a pod from this list, and assigns them to a 'suitable' node, after which that node's Kubelet service will then take over and initialize the pod.

How does the kube-scheduler work out which node is 'suitable'? The kube-scheduler performs a series of checks to identify a list of suitable nodes. A node needs to pass all these checks before being deemed suitable:

### 1. Whether the node has scheduling enabled 

which is false if the node has been drained:

```bash
root@kube-master:~# kubectl drain kube-worker2 --ignore-daemonsets
node/kube-worker2 drained

root@kube-master:~# kubectl get nodes
NAME           STATUS                     ROLES    AGE   VERSION
kube-master    Ready                      master   43h   v1.14.1
kube-worker1   Ready                      <none>   43h   v1.14.1
kube-worker2   Ready,SchedulingDisabled   <none>   43h   v1.14.1
```

### 2. Whether the node has enough resource capacity

available to meet the pods resource requirements. 


This is whether a nod has enough ram+cpu to satify a pods resource quota, i.e. a pod's `pod.spec.containers.resources` requirements:

```yaml
...
    spec: 
      containers:
        - name: cntr-centos
          image: centos:latest
          resources:             # requests are used for minimum requirements. 
            requests:
              memory: "64Mi"
              cpu: "100m"    # 1000m = 1 cpu core. So here we're requesting just 10%.
            limits:
              memory: "128Mi"
              cpu: "200m
...
```

### 3. NodeSelector settings

This means a node is suitable if it has label's that matches the pod's `pod.spec.nodeSelector` setting. e.g. if the pod spec has:

```yaml
---
...
spec:
  nodeSelector:           # add this line
    ec2InstanceType: M3     # specify the node label to match with.
  containers:
    - name: cntr-httpd
      image: httpd:latest 
      ports:
        - containerPort: 80
...
```

Then you can make a node meeth this requirement by running:

```bash
$ kubectl label nodes kube-worker2 ec2InstanceType=M3
node/kube-worker2 labeled
```

### 4. Node Taints and Pod Tolerations

If pod has tolerations to override the taints. 


### 5. Node Affinity


### 6. Pod Affinity


### 7. Pod Anti-Affinity

### 8. HostPorts

This is the `pods.spec.containers.ports.hostPort`. Basically a pod listens on a particular port number at the worker node level. Covered later



## Resulting matches

Once all the checks are done, the kube-scheduler will end up with a list of nodes that passed the checks. Here's a few pottential scenarios:

1. **No nodes passed the checks** - in that case the pod doesn't get deployed and shows as 'pending'. The scheduler also doesnt re-attempt to assign the pod since it has already been taken off the queue. 
2. Multiple nodes passed the checks - in that case the kue-schedular deploys to nodes with best on best match, and if all nodes are equally good matches, then deploy pods using round robin (if we're talking about controller-provisioned pods, such as replicasets and deployments). 

## Exceptions to the rule

Kube-schedule do not procision daemonsets or statefulsets controlled pods.

## References

[https://medium.com/@dominik.tornow/the-kubernetes-scheduler-cd429abac02f](https://medium.com/@dominik.tornow/the-kubernetes-scheduler-cd429abac02f)

[https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)

[https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/](https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/)
