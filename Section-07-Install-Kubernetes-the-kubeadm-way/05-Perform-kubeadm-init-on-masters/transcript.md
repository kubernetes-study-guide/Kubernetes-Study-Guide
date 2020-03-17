Ok let's try the install:

```
kubeadm init --control-plane-endpoint "188.166.136.150:6443" --upload-certs
```

Not recommended to use ip number. so need to fix that. 


Also preflight failed, so had to do this:

```
vi /etc/sysctl.conf
net.bridge.bridge-nf-call-iptables = 1
sysctl -p
```

If you have problems like that, then google is your friend.


This can take a few minutes to run. The output will be something like:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 188.166.136.150:6443 --token sqry6c.lozgis6l2iqhszxc \
    --discovery-token-ca-cert-hash sha256:042b8093c2206e8c9514e60431ccb04adb433c663a2ca0d2963fa8dc6b604540 \
    --control-plane --certificate-key c47a50963a0cbce501f299cc2625a44886872dcfe08e97ece987f5342706ce6a

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 188.166.136.150:6443 --token sqry6c.lozgis6l2iqhszxc \
    --discovery-token-ca-cert-hash sha256:042b8093c2206e8c9514e60431ccb04adb433c663a2ca0d2963fa8dc6b604540\
```






Now at the moment kubectl hasn't been configured to use this cluster. 


```
[root@kube-master-1 ~]# kubectl cluster-info
The connection to the server localhost:8080 was refused - did you specify the right host or port?

```

So let's follow these instructions. 

Then try again:

```
kubectl get nodes -o wide
```

This time it worked. 


```
[root@kube-master-1 ~]# kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}


[root@kube-master-1 ~]# kubectl cluster-info
Kubernetes master is running at https://10.131.127.159:6443
KubeDNS is running at https://10.131.127.159:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
[root@kube-master-1 ~]#
```


```
[root@kube-master-1 ~]# kubectl get nodes -o wide
NAME            STATUS     ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
kube-master-1   NotReady   master   9m10s   v1.17.4   10.131.127.159   <none>        CentOS Linux 7 (Core)   3.10.0-957.27.2.el7.x86_64   docker://19.3.8
```

This looks better, but it's showing not ready. To fixy that we need to install a plugin:

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

and wait for it to come up:

```
kubectl get pod -n kube-system -w
```




The output also shows what command to run on the worker nodes to get them added to this master. In case you lose it, you can print it out again:


```
kubeadm token create --print-join-command
```


Now let's create our other 2 masters:


https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/


when you run the join command you'll end up with output like:


```
This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane (master) label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
* A new etcd member was added to the local/stacked etcd cluster.

To start administering your cluster from this node, you need to run the following as a regular user:

	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.

```

After that you should end up with:

```
[root@kubermaster1 ~]# kubectl get nodes -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
kubemaster2    Ready    master   8m17s   v1.17.4   10.131.125.53    <none>        CentOS Linux 7 (Core)   3.10.0-957.27.2.el7.x86_64   docker://19.3.8
kubemaster3    Ready    master   5m19s   v1.17.4   10.131.127.176   <none>        CentOS Linux 7 (Core)   3.10.0-957.27.2.el7.x86_64   docker://19.3.8
kubermaster1   Ready    master   52m     v1.17.4   10.131.122.38    <none>        CentOS Linux 7 (Core)   3.10.0-957.27.2.el7.x86_64   docker://19.3.8
```