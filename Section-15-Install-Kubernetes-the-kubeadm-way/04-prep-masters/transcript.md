1. log into your first master:

I already have my private key loaded in:

```
$ ssh-add -l
```

So let's try logging in as root:

```
$ ssh root@134.122.104.96
```

Now let's confirm that I have centos8:


```
hostnamectl
```



Now install kubeadm:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

In particular follow:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

No need to do `yum update` - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#installing-kubeadm-on-your-hosts because just did the install now. 


After that you should end up with:


```
kubeadm version --output short
v1.17.4

kubectl version --short --client
Client Version: v1.17.4

kubelet --version
Kubernetes v1.17.4
```

Now we need to install docker:
https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

The install command is:

```
yum install docker-ce docker-ce-cli containerd.io
```




Now start docker:

```
systemctl start docker
systemctl enable docker
systemctl status docker
```


Let's check the version:

```
[root@kubemaster2 ~]# docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b
 Built:             Wed Mar 11 01:27:04 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.17
  Git commit:       afacb8b
  Built:            Wed Mar 11 01:25:42 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

In docker, This version 19 means that this version of docker was released in the year 2019. and the next part 03, means it was released in the 3rd quarter of 2019. 


We're now ready to perform the actuall install. Let's do that in the next video.


Also had to do this:

```
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
sysctl -p
```
Otherwise preflight check fails. you might have other issues. Google is your friend. 


You have to do all of the above steps in the two masters. 