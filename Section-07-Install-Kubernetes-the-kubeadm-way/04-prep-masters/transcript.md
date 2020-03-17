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

Now lets start with a yum update, just to get everything updated:

```
yum update
```


Now install kubeadm:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

In particular follow:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

No need to do `yum update` - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#installing-kubeadm-on-your-hosts because just did the install now. 


After that you should end up with:


```
$ kubeadm version --output short
v1.17.4

$ kubectl version --short --client
Client Version: v1.17.4

$ kubelet --version
Kubernetes v1.17.4
```

Ok let me show you what happens when I try init. It will fail and you'll see why:

```
kubeadm init
```

ok that failed because it couldnt find docker. 

Let's install that now:

```
yum list available | grep docker
```

This looks like a really old version of docker. we should look for docker 19, for 2019. 

So let's try installing newer version of docker by following the docs:
https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository

The install command is:

```
yum install docker-ce docker-ce-cli containerd.io
```

In docker, This version 19 means that this version of docker was released in the year 2019. and the next part 03, means it was released in the 3rd quarter of 2019. 

Let's check the version:

```
 docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b
 Built:             Wed Mar 11 01:27:04 2020
 OS/Arch:           linux/amd64
 Experimental:      false
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```


Now start docker:

```
[root@kubermaster1 ~]# systemctl start docker
[root@kubermaster1 ~]# systemctl status docker
```

Now we're ready to perform the actuall install. Let's do that in the next video.

