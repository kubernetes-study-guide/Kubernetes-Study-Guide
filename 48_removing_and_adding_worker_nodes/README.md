# Removing worker nodes

There will be times where you want to disable a node temporarily to do a bit of maintenance, for example the worker node has a very old OS and you want to perform an OS upgrade. 

If you just turn off the worker node, the kubecluster will assume it just temporarily lost connection and will wait 5 minutes before it spins up new pods to replace the lost pods. That 5 minute delay can cause disruptions. That's why it's important to take nodes offline in a safe way. 


## Temporarily taking a worker node offline

Let's say we have the following cluster:

```bash
$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
kube-master    Ready    master   33m   v1.14.1
kube-worker1   Ready    <none>   31m   v1.14.1
kube-worker2   Ready    <none>   55s   v1.14.1
```

And this cluster has the following pods running:

```bash
$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
dep-caddy-856475bcb9-ghsdc   1/1     Running   0          3m37s   192.168.1.3   kube-worker1   <none>           <none>
dep-caddy-856475bcb9-tbl9r   1/1     Running   0          3m37s   192.168.2.3   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-sbv4r   1/1     Running   0          4m36s   192.168.2.2   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-wfw72   1/1     Running   0          4m36s   192.168.1.2   kube-worker1   <none>           <none>
ds-httpd-9cxng               1/1     Running   0          19s     192.168.2.4   kube-worker2   <none>           <none>
ds-httpd-jbrn4               1/1     Running   0          19s     192.168.1.4   kube-worker1   <none>           <none>
```


Here we have 6 pods running across 2 worker nodes, kube-worker1 and kube-worker2. The last 2 pods in the lease, are actually deamonset pods:

```bash
$ kubectl get daemonsets
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-httpd   2         2         2       2            2           <none>          2m45s
```



In our example we want to disable kube-worker1. before making a node offline. The best way to do this is to move all pods out of kube-worker1 and place them in the other worker nodes, i.e. we want to drain kube-worker1:
 
```bash
$ kubectl drain kube-worker1 --ignore-daemonsets
node/kube-worker1 cordoned
WARNING: ignoring DaemonSet-managed Pods: default/ds-httpd-jbrn4, kube-system/calico-node-4qcf7, kube-system/kube-proxy-qxb8f
evicting pod "dep-httpd-8678c9bdd8-wfw72"
evicting pod "dep-caddy-856475bcb9-ghsdc"
pod/dep-httpd-8678c9bdd8-wfw72 evicted
pod/dep-caddy-856475bcb9-ghsdc evicted
node/kube-worker1 evicted
```

This results in all the pods being moved out of kube-worker1, and placed on the remaining worker nodes, which in this case is just kube-worker2. 

```bash
$ kubectl get oids -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
dep-caddy-856475bcb9-mbdgt   1/1     Running   0          3m12s   192.168.2.5   kube-worker2   <none>           <none>
dep-caddy-856475bcb9-tbl9r   1/1     Running   0          12m     192.168.2.3   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-s5scv   1/1     Running   0          3m12s   192.168.2.6   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-sbv4r   1/1     Running   0          13m     192.168.2.2   kube-worker2   <none>           <none>
ds-httpd-9cxng               1/1     Running   0          9m17s   192.168.2.4   kube-worker2   <none>           <none>
ds-httpd-jbrn4               1/1     Running   0          9m17s   192.168.1.4   kube-worker1   <none>           <none>
```
The only exception being one of the daemonset pods, since we used the --ignore-daemonsets flag. That's because we shouldn't have more than one deamonset pod (from the same daemonset) running on a worker node. If we view our nodes again we should see that scheduling of pods to kube-worker1 has been disabled:

```bash
$ kubectl get nodes -o wide
NAME           STATUS                     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kube-master    Ready                      master   53m   v1.14.1   10.2.5.110    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker1   Ready,SchedulingDisabled   <none>   51m   v1.14.1   10.2.5.111    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
kube-worker2   Ready                      <none>   20m   v1.14.1   10.2.5.112    <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1
```

Now it's safe to poweroff and/or do maintenance work on kube-worker1. Once kube-worker1 node is ready to be added back to the kubecluster, you just to need to run:

```bash
$ kubectl uncordon kube-worker1
node/kube-worker1 uncordoned
```

However this doesn't even redistribute the pods again, it just means that in future new pods can be schedules on the now online worker node:

```bash
root@kube-master:~/kubernetes-study-guide/14_DaemonSet/configs# kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
dep-caddy-856475bcb9-mbdgt   1/1     Running   0          17m   192.168.2.5   kube-worker2   <none>           <none>
dep-caddy-856475bcb9-tbl9r   1/1     Running   0          26m   192.168.2.3   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-s5scv   1/1     Running   0          17m   192.168.2.6   kube-worker2   <none>           <none>
dep-httpd-8678c9bdd8-sbv4r   1/1     Running   0          27m   192.168.2.2   kube-worker2   <none>           <none>
ds-httpd-9cxng               1/1     Running   0          23m   192.168.2.4   kube-worker2   <none>           <none>
ds-httpd-jbrn4               1/1     Running   0          23m   192.168.1.4   kube-worker1   <none>           <none>
```

## Permenantly delete a worker node

You may want delete a worker node permanently, e.g. you have too many nodes and not enough pods, in which case may want to delete a node to save money. In that case, to do it safely, first drain the node before deleting it:

```bash
# kubectl drain kube-worker1 --ignore-daemonsets
# kubectl delete nodes kube-worker1
node "kube-worker1" deleted
```


## Adding new worker nodes

You might find that you need to add new worker nodes because you're running low on capacity. If that's the case, just run the [print-to-join](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/#token-based-discovery-with-ca-pinning) command on the master:

```bash
kubeadm token create --print-join-command
kubeadm join 10.2.5.110:6443 --token 8qo20w.05p788jbh8z59cvm     --discovery-token-ca-cert-hash sha256:86dc21e3ca348f695d5aabe534a568ae3e3e68b5bc42073751663b6863855554
```

Then run this output on your new worker node. 



