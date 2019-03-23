# Daemonsets

This is a variation of deployment object, but in the case of daeamonsets, it ensures exactly one pod is runnnig per worker node. This means:

replicas = no of worker nodes in the kubecluster.

This means that the number of deamonset pods goes up/down every time you add/remove worker nodes.

There are a few use-cases for these:

- log aggregators - Have one filebeat pod per worker node, to collect logs of all other pods
- monitor - a pod that monitors the worker node itself. 
- loadbalancers/Rev proxies/API Gateways



Daemonsets are also used by the kubecluster internally:


```bash
root@kube-master:~# kubectl get nodes -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kube-master    Ready    master   6m50s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker1   Ready    <none>   4m24s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker2   Ready    <none>   2m56s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1

root@kube-master:~# kubectl get daemonsets --namespace=kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
calico-node   3         3         3       3            3           beta.kubernetes.io/os=linux   4m
kube-proxy    3         3         3       3            3           <none>                        4m45s



root@kube-master:~# kubectl get pods -o wide --namespace=kube-system | grep proxy
kube-proxy-bnzr6                      1/1     Running   0          9m48s   10.0.2.15     kube-master    <none>           <none>
kube-proxy-hcs4t                      1/1     Running   0          6m13s   10.0.2.15     kube-worker2   <none>           <none>
kube-proxy-p7kpl                      1/1     Running   0          7m40s   10.0.2.15     kube-worker1   <none>           <none>
root@kube-master:~# kubectl get pods -o wide --namespace=kube-system | grep calico
calico-node-8q45b                     2/2     Running   0          6m15s   10.0.2.15     kube-worker2   <none>           <none>
calico-node-skhmr                     2/2     Running   0          7m42s   10.0.2.15     kube-worker1   <none>           <none>
calico-node-xrptm                     2/2     Running   0          9m19s   10.0.2.15     kube-master    <none>           <none>
```

Since minikube is only a single node cluster, I have created a [kubernetes vagrant project](https://github.com/Sher-Chowdhury/kubernetes-the-kubeadm-way-vagrant) that spins up a 3-node kube cluster, 1 master and 2 workers: 


```bash
root@kube-master:~# kubectl get nodes -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kube-master    Ready    master   8m10s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker1   Ready    <none>   4m28s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker2   Ready    <none>   2m59s   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
```

The daemonset yaml file looks like this, it looks a lot like a replicaset:


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
      containers:
        - name: cntr-httpd
          image: httpd:latest 
          ports:
            - containerPort: 80
```





Since our kubecluster has 2 worker nodes, it results in 2 pods being created even though we didn't specify any replicas:

```yaml
# kubectl get ds -o wide
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE   CONTAINERS   IMAGES         SELECTOR
ds-httpd   2         2         2       2            2           <none>          47s   cntr-httpd   httpd:latest   app=httpd_webserver

# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
ds-httpd-jkc6r   1/1     Running   0          62s   192.168.1.2   kube-worker1   <none>           <none>
ds-httpd-mtqcc   1/1     Running   0          62s   192.168.2.2   kube-worker2   <none>           <none>

```





# Deploying DaemonSet to some nodes using nodeSelector

You may want to deploy a particular daemonset on a subset of worker nodes, e.g. all nodes of ec2 instance type of M3. You can do that by assigning arbitrary labels to your pods:


```bash
$ kubectl label nodes kube-worker2 ec2InstanceType=M3
node/kube-worker2 labeled

$ kubectl get nodes --show-labels
NAME           STATUS   ROLES    AGE   VERSION   LABELS
kube-master    Ready    master   81m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-master,node-role.kubernetes.io/master=
kube-worker1   Ready    <none>   77m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-worker1
kube-worker2   Ready    <none>   75m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ec2InstanceType=M3,kubernetes.io/hostname=kube-worker2

```

Then use the ds.spec.template.spec.nodeSelector setting in your yaml file to apply the restriction:

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

This results in:


```bash
# kubectl get daemonsets
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR        AGE
ds-httpd   1         1         1       1            1           ec2InstanceType=M3   22s
# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
ds-httpd-h7t6q   1/1     Running   0          26s   192.168.2.3   kube-worker2   <none>           <none>
```

Note, the nodeSelector is specified at the pod-spec level. That means you can use nodeSelector in lots of other objects, e.g. pods, replicaSets, Deployments,...etc. There are other ways to [assign pods to particular worker nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/).

