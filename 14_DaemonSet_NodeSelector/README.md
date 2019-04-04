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

