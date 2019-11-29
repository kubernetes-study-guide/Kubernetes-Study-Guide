Hello everyone, and welcome back.

Ok, a little while ago, we saw how you can do pod-to-pod communication using IP addresses. We're going to do the same thing again, but this time using DNS names instead of ip addresses. To do this we need to make use of Kubernete's own internal DNS service, which is called Kubernetes DNS. Kubernetes DNS is actually an addon that you can install into your cluster. Luckily Kubernetes DNS comes installed by default if you use one of the automated kube cluster provisioning tools, such as minikube and kubeadm. 

Ok before we show how kubernetes DNS works, let me do a quick recap on the ip address approach. 

Here I've opened up a bash terminal inside this video's topic folder. 

```
pwd
```


So let's create the same 2 pods that we used earlier. 

```
$ pwd
$ code configs/pod-centos.yml configs/pod-httpd.yml
$ kubectl apply -f configs/pod-httpd.yml -f configs/pod-centos.yml
$ kubectl get pods -o wide
$ kubectl exec pod-centos -- curl --silent http://172.17.0.9
```
==== keep using centos, since centos image now comes with dnf. 
==== introduce ubi as part of docker-registry video. 
By the way you may have noticed that in my earlier demos I used the centos Docker image to create my dummy test pod. However I've now switched to using the Universal Base Image, or UBI for short. The UBI image is an enterprise grade image developed by redhat. And the best part is that this image is available for free. UBI is not available on docker hub though, instead you need to download it from redhat's own registry. That's why I needed to specify the full image path. If you just specify the name, then kubernetes will default to prefixing the dockerhub's registry, to the image name behind the scenes. The docker hub's registry url is docker.io. For example if you specify busybox, then kubernetes will assume you meant docker.io/busybox. I'll cover more about using third party docker registries later in this course. 
====

So that's how far we got to last time, we managed to get one pod talking to another pod using an ip address. Now in this demo we still want to run this curl command, but this time using a DNS name rather than an IP address.


So to start using DNS, we first need to create a service object. So here's the service yaml file we'll use to do that:

```
code config/svc-nodeport-httpd.yaml
```

I'm going to close the centos tab to free up some screen space. 

I don't want to get sidetracked by going over everything in this service file just yet. Instead I'll go through it in the next video. For now, the only thing you need to know is that this service uses label&selectors to associate itself with this pod, and this service will accept internal traffic from port xxxxx and forward it to port xxxx, which is the port our apache pod is listening on. 


So let's go ahead and create this service.

```
$ kubectl apply -f configs/svc-nodeport-httpd.yml
$ kubectl get services -o wide
```

The important thing to note here is that, as part of creating this service, kubernetes took this service's name, along with it's ip address, and used them to register a new dns record in Kubernetes DNS. We can confirm that by doing an nslookup inside our centos test pod:

```
$ kubectl exec -it pod-centos -- bash
$ nslookup svc-nodeport-httpd
```

Ok it looks like nslookup isn't installed, so let's go ahead install it.


```
dnf whatprovides */nslookup
```

Here we can see that nslookup comes as part of the bind-utils package. So let's install that:

```
dnf install -y bind-utils
```

Ok nslookup should now be instealled. Let's now try using nslookup again:

```
nslookup svc-nodeport-httpd
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   svc-nodeport-httpd.default.svc.cluster.local
Address: 10.105.186.109
```

As you can see, we now have a new dns record, which is the fqdn of our service along with it's ip address. 


Now, finally, let's try out our new dns record:


```
[root@pod-centos /]# curl http://svc-nodeport-httpd.default.svc.cluster.local:3050
<html><body><h1>It works!</h1></body></html>
```

Awesome that worked! That means that we no longer need to rely on ip addresses as long as we use service objects and their DNS entries. This effectively means that services are acting as a gateway to our pods.

Now let's take a closer look at the url we used in our test. You might have noticed that I used port 3050 here. That's because in the service spec we said that this service can only accept internal traffic on this port.


The 'default' in the fqdn actually refers to the namespace that the service live's in. 

And since our centOS pod happens to also live in the same namespace as the service itself, it means we can get away with just curling the service's name:

```
$ curl http://svc-nodeport-httpd:3050
<html><body><h1>It works!</h1></body></html>
```

As you can see this still worked. That's because the resolv.conf has a default basename that get's used as the default if you don't explicitly specify one in the url. 

```
$ cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

The resolv.conf also contains the nameserver setting. This specifies the ip address for the pod to use to perform DNS queries against Kubernetes DNS. This ip address is something that kubernetes has automatically inserted into the resolv.conf at the time of creating this pod. So let's see where this ip address leads to:
```
$ kubectl get all -o wide --all-namespaces | grep 10.96.0.10
kube-system            service/kube-dns                    ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   4h34m   k8s-app=kube-dns
```

Here we find that it belongs to a service called kube-dns. This service lives in the kube-system namespace, and is configured to listen on port 53, which is that standard port for DNS traffic. This service forwards any DNS traffic traffic to pods with this label (redbox).

And if we look at what pods has this label, we find they are coredns pods. 


```
$ kubectl get pods --namespace=kube-system -l k8s-app=kube-dns --show-labels
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-5wm6q   1/1     Running   0          4h45m
coredns-5644d7b6d9-kzpjj   1/1     Running   0          4h45m
```

That's because, under the covers, Kubernetes DNS is actually built on top of another open source project called, CoreDNS. These pods are responsible for providing the core DNS service to the cluster. So whenever we create a service object, a new dns record gets added to these pod's internal dns database. 

Also if coredns receives a dns lookup request that it's unfamiliar with, such as codingbee.net, then coredns will seek help from one of the dns servers on the wider internet to resolve the lookup request for them. 

```
$ kubectl exec xxxxxx -- dig codingbee.net
```

That's it for this video. I'll see you in the next one. 



