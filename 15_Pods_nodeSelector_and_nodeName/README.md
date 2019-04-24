# Pod nodeSelector and nodeName


## nodeSelector
You may want to deploy your pods to a particular subset of worker nodes, e.g. all nodes of ec2 instance type of M3. You can do that by assigning arbitrary labels to your nodes:


```bash
$ kubectl label nodes kube-worker2 ec2InstanceType=M3
node/kube-worker2 labeled

$ kubectl get nodes --show-labels
NAME           STATUS   ROLES    AGE   VERSION   LABELS
kube-master    Ready    master   81m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-master,node-role.kubernetes.io/master=
kube-worker1   Ready    <none>   77m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-worker1
kube-worker2   Ready    <none>   75m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ec2InstanceType=M3,kubernetes.io/hostname=kube-worker2

```

Then use the ``pod.spec.nodeSelector`` setting in your yaml file to apply the restriction:

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-httpd
  labels:
    app: httpd_webserver
spec:
  selector:
    matchLabels:
      app: httpd_webserver
  template:
    metadata:
      labels:
        app: httpd_webserver
    spec:
      nodeSelector:           # add this line
        ec2InstanceType: M3     # specify the node label to match with.
      containers:
        - name: cntr-httpd
          image: httpd:latest 
          ports:
            - containerPort: 80
```

We used a daemonset here to show that you can override a deamonset's default behaviour. This results in:


```bash
$ kubectl get daemonsets
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR        AGE
ds-httpd   1         1         1       1            1           ec2InstanceType=M3   22s


$ kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
ds-httpd-h7t6q   1/1     Running   0          26s   192.168.2.3   kube-worker2   <none>           <none>
```

Note, the nodeSelector is specified at the pod-spec level. That means you can use nodeSelector in lots of other objects, e.g. pods, replicaSets, Deployments,...etc. There are other ways to [assign pods to particular worker nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/).



## nodeName

The ``pod.spec.nodeName`` setting works in a very similar way, but this setting restricts pods to be deployed on single worker node, e.g. to deploy all my deployment's pods to kube-worker2:

```bash
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kube-master    Ready    master   8m33s   v1.14.1   10.2.5.110    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker1   Ready    <none>   6m      v1.14.1   10.2.5.111    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker2   Ready    <none>   4m33s   v1.14.1   10.2.5.112    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
```

I would use this setting:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-httpd
  labels:
    app: httpd_webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd_webserver
  template:
    metadata:
      labels:
        app: httpd_webserver
    spec:
      nodeName: kube-worker2      # Added this line
      containers:
        - name: cntr-httpd
          image: httpd:latest 
          ports:
            - containerPort: 80
```

This results in:

```bash
# kubectl get pods -o wide
NAME                         READY   STATUS              RESTARTS   AGE   IP       NODE           NOMINATED NODE   READINESS GATES
dep-httpd-6c9c5c4984-drjfc   0/1     ContainerCreating   0          11s   <none>   kube-worker2   <none>           <none>
dep-httpd-6c9c5c4984-hm8rv   0/1     ContainerCreating   0          11s   <none>   kube-worker2   <none>           <none>
dep-httpd-6c9c5c4984-kptzb   0/1     ContainerCreating   0          11s   <none>   kube-worker2   <none>           <none>
```




This setting essentially overrides the kube-scheduler and let's you manually assign pods to nodes. 
