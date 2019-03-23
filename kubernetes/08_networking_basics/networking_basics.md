# Networking Basics

In Kubernetes, 'services' is actually all about networking. In Docker world, when you use docker-compose, all the networking is done for you automatically behind the scenes. With Kubernetes on the other hand, you will need to set up a lot of the networking. However there are some basic networking features that comes out-of-the-box with Kubernetes:

- A pod's internal networking
- IP based pod-to-pod networking
- IP based kube-node-to-pod networking
- dns-service (kube-dns)


  
  
## A pod's internal networking (eg1-networking-inside-pods)

If you have 2+ containers inside a single pod, then these containers can reach each other via localhost. Let's say we create the following pod:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo      # both container's hostname will be set to this
  labels:
    component: centos
spec:
  containers:
    - name: cntr-httpd
      image: httpd:latest
      ports:
        - containerPort: 80
    - name: cntr-centos
      image: centos
      command: ["/bin/bash", "-c"]
      args:
        - |
          while true ; do
            date
            curl http://localhost
            sleep 10
          done
```

This defines a single pod that's housing 2 containers (and in our case kubernetes has auto-assigned an IP of 172.17.0.7 to the pod):

```bash
$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-demo   2/2     Running   0          28m   172.17.0.7   minikube   <none>           <none>
```

These containers should be able to talk to each other without needing any further configurations. Let's start by logging into one of the containers:

```bash
$ kubectl exec pod-demo -c cntr-centos  -it /bin/bash
[root@pod-demo /]#

```

Notice that I also specified the -c (container) flag followed by the container name. That's only required when logging into a container in a multi-container pod, so that kubectl knows which container you want to access. If you omit this then kubectl will default to logging into the first container that's listed in the yaml file. With multi-container pods it's really important to only have on primary container, and all other containers act as secondary/supporting/sidecar containers. Meaning that, if the sidecar containers fails, then the pod's main app (primary container) still continues to function.



Also notice that the containers hostname is the same as the pod's name:

```bash
[root@pod-demo /]# hostname
pod-demo
```

All containers in a pod are actually assigned the same hostname, which is the pod's hostname. You'll find is that each container is attached to 2 network interfaces:

```bash
[root@pod-centos /]# yum install -q -y net-tools      # we have to install ifconfig before we can use it. 

[root@pod-centos /]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.7  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:07  txqueuelen 0  (Ethernet)
        RX packets 3475  bytes 9754927 (9.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2819  bytes 161157 (157.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 40  bytes 3516 (3.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 40  bytes 3516 (3.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

These are virtual network interfaces that exists as part of the pod. These same network interfaces are actually attached to all the containers in the pod at the same time. Because of that, it means that, from cntr-centos container you can reach the cntr-httpd via either of these interfaces ip address:

```bash
[root@pod-centos /]# curl http://127.0.0.1
<html><body><h1>It works!</h1></body></html>
[root@pod-centos /]# curl http://172.17.0.7
<html><body><h1>It works!</h1></body></html>
```

Here our pod routes this curl request internally to the cntr-httpd container.

**Question:** If our pod had several other containers, then how would our pod know which container to route the curl request to? 
**Answer:** Our yaml file specifies which port each container is listening on (if any). Therefore our pod knows that requests destined to port 80 should be routed to cntr-httpd container.



You also find that the following works as well:

```bash
[root@pod-centos /]# curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

That's because all the containers in the pods has the following hosts file, which resolves 'localhost' to it's internal network interface, 127.0.0.1:

```bash
[root@pod-centos /]# cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
172.17.0.7      pod-centos
```

Furthermore all containers in a pod, have the same hostname, which by default is the pod's name:

```bash
[root@pod-centos /]# hostname
pod-centos
```

and since we have an entry in the hosts file for this hostname, it means the following also works:

```bash
[root@pod-centos /]# curl http://pod-centos
<html><body><h1>It works!</h1></body></html>
```

In both 'locahost' and 'hostname' curl requests, a static internal dns-lookup occured thanks to the /etc/hosts entry.

So far we did a **cntr-centos**-->**cntr-httpd**. But we didn't do it in the other direction. That's because cntr-centos is in theory reachable, but we haven't setup anytihng (e.g. web servre) on cntr-centos to listen on any ports.



## IP based Pod-2-Pod networking

[Pod-2-pod](https://kubernetes.io/docs/concepts/cluster-administration/networking/) is possible thanks to the fact that Kubernetes auto-assigns an private ip address to all pods during the pod's launch time. The pods can reach each other with these ip addresses.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
spec:
  containers:
    - name: cntr-httpd
      image: httpd:latest
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
spec:
  containers:
    - name: cntr-httpd
      image: nginx:latest
      ports:
        - containerPort: 80
```

The pod's ip are shown below.

```bash
$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-httpd   1/1     Running   0          54s   172.17.0.8   minikube   <none>           <none>
pod-nginx   1/1     Running   0          54s   172.17.0.7   minikube   <none>           <none>
```

To test this, we need to log into one of the pods, and run curl to the other pod. Let's log into pod-httpd:

```bash
kubectl exec pod-httpd -it /bin/bash
root@pod-httpd:/usr/local/apache2# apt-get update
root@pod-httpd:/usr/local/apache2# apt-get install -y curl
```




We also had to install the curl command itself. Before we can perform the test:


```bash
root@pod-httpd:/usr/local/apache2# curl http://172.17.0.7
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

However we can't rely on IP addresses because they are prone to changing, e.g. when a pod is rebuilt. So hard coding ip addresses in various places is not an option. The conventional to address this problem is giving each pod a human-readable dns name, which is kept up to date dynamically, that's where kube-dns comes to the rescue


## DNS Service

Minikube comes with a builtin internal dns service, kube-dns ([soon to be coredn](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)).


```bash
$ kubectl get svc kube-dns --namespace=kube-system -o wide
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE    SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   3d3h   k8s-app=kube-dns
```

This ip address is automatically added to all other pods /etc/resolv.conf file:

```bash
root@pod-nginx:/# cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

So all we need to do is create dns entries in our internal dns server (kube-dns). That's done by creating service objects. There are different types of service objects, such as nodePort, and ClusterIP service. We'll cover them later. 


## The 'kubectl expose' command

In this course we'll focus on creating service objects declaratively using yaml files. But you can create them implicitly from the command line usng the 'expose' subcommand:

```bash
kubectl expose pod podname --type=NodePort --name servicename
```