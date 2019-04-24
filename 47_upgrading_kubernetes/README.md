# Upgrading Kubernetes


> This article uses the following repo as a demo (using kubeadm-upgrade-demo branch): https://github.com/Sher-Chowdhury/kubernetes-the-kubeadm-way-vagrant/tree/kubeadm-upgrade-demo

Newer versions Kubernetes are released every few months. So it's best to stay up to date by regularly upgrading your kube clusters. If you provisioned your cluster with kubeadm, then you can use also use kubeadm to perform the upgrades too. For this article we'll assume that kubeadm.

Let's first check what version we have of everything. We have the following version of kubelet:

```bash
$ kubectl version --short
Client Version: v1.13.5
Server Version: v1.13.5

# Thee above doesn't specify kubelet version, for the kubelet version you need to run:
$ kubectl get nodes
NAME           STATUS   ROLES    AGE    VERSION
kube-master    Ready    master   102m   v1.14.0
kube-worker1   Ready    <none>   100m   v1.13.5


# check what version of kubeadm we have on the kubemaster
root@kube-master:~# kubeadm version -o short
v1.13.5
```

## Upgrading using the apt-get route

### upgrade master
First you have to find out whether kubeadm has been installed by apt-get:

```bash
root@kube-master:~# apt list --installed | grep kubeadm
kubeadm/kubernetes-xenial,now 1.14.0-00 amd64 [installed,upgradable to: 1.14.1-00]
```
If this doesn't show anything, then it indicates that it has been installed dy downlaoding the binary. 




Now you need to choose what version to upgrade to by choosing which version of kubeadm to install:

```bash 
$ apt-cache madison kubeadm
   kubeadm |  1.14.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
...

```


Note, if this list doesn't appear, then you need to create the `/etc/apt/sources.list.d/kubernetes.list` and then refresh local cache, as described here [https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) 

In our case we'll upgrade kuberentes to 1.14.0, so we upgrade kubeadm to that version too:

```bash
root@kube-master:~# apt-get install --only-upgrade kubeadm=1.14.0-00
...

root@kube-master:~# kubeadm version -o short
v1.14.0
```

Now let's run the following as a dry run:

```bash
root@kube-master:~# kubeadm upgrade plan v1.14.0
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.13.5
[upgrade/versions] kubeadm version: v1.14.0

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     2 x v1.13.5   v1.14.0

Upgrade to the latest version in the v1.13 series:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.13.5   v1.14.0
Controller Manager   v1.13.5   v1.14.0
Scheduler            v1.13.5   v1.14.0
Kube Proxy           v1.13.5   v1.14.0
CoreDNS              1.2.6     1.3.1
Etcd                 3.2.24    3.3.10

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.14.0

_____________________________________________________________________
```


This shows what's currently installed and what can be upraded to. Notice that the output says tht the kubelet component needs to be upgraded seperately, we'll show how to do that further below. This also shows what command to run to perform the actual upgrade:


```bash
root@kube-master:~# kubeadm upgrade apply v1.14.0
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
...
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.14.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

This has now done the upgrade:

```bash
# kubectl version --short
Client Version: v1.13.5
Server Version: v1.14.0
```

We'll also need to upgrade our local kubectl command too. e.g. if you installed kubectl on your macbook using brew, then uprade it with brew as well.



### Upgrade the kubelet component
As indicated, this upgrade process hasn't upgraded the kubelet component, which is still at the old version:

```bash
# kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
kube-master    Ready    master   79m   v1.13.5
kube-worker1   Ready    <none>   77m   v1.13.5
```

This has to be done manually, by running the following on every kube masters and nodes:

```bash
apt-get install --only-upgrade kubelet=1.14.0-00
```

Then restart the kubelet daemon:

```bash
root@kube-master:~# systemctl restart kubelet

root@kube-master:~# kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
kube-master    Ready    master   84m   v1.14.0
kube-worker1   Ready    <none>   82m   v1.13.5
```

## Upgrading by downloading the binary

First choose which version you want to install by selecting the changelog file. 

E.g. we want to upgrade to version 1.14. So we choose the [CHANGELOG-1.14.md](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md). Then select the patch version, in our case we want, [v1.14.0](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#v1140). This has the client binaries (which has kubectl binary) and server binaries (which has the kubeadm, kubelet binary...etc). Next download these binaries using curl, and then extract the tar file:

```bash
curl -SL https://dl.k8s.io/v1.14.0/kubernetes-server-linux-amd64.tar.gz -o kubernetes-server-linux-amd64.tar.gz
tar -xvzf kubernetes-server-linux-amd64.tar.gz
```

Now replace existing kubeadm binary with this new binary. You can then follow the rest of the instruction. 
