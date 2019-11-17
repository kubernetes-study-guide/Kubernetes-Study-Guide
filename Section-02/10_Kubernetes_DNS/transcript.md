Hello everyone one, and welcome back.

Earlier we saw how you can do pod-to-pod communication using pod IP addresses. This time we're going to do the same thing but using DNS names rather than IP addresses. That's possible thanks to Kubernete's own internal DNS service, which is called Kubernetes DNS. Kubernetes DNS is an addon that comes preinstalled by default in most installer options, including a minikube provisioned cluster. Kubernetes DNS is actually built on top of another open source software called CoreDNS.

Ok before we show how kubernetes DNS works, let's first remind ourselves on how to do pod-to-pod communication using ip addresses:

```
code configs/pod-centos.yml
code configs/pod-httpd.yml
```

These are the same pod yaml definitions I used before. So let's create these 2 pods

```
$ kubectl apply -f configs/pod-centos.yml -f configs/pod-httpd.yml
$ kubectl get pods -o wide
$ kubectl exec pod-centos -- curl --silent http://172.17.0.9
```

So that's how far we got last time. Now to start using DNS, we need to create a service object. Service objects are used for intelligently forwarding traffic to pods.

So here's the service we'll create:

```
code config/svc-nodeport-httpd.yaml
```


Here we're saying:

- We want to create a service object 
- This service's name is going to be svc-nodeport-httpd
- this service is going to be of the type nodeport nodeport. There are other service types, and we'll cover them later. 

- Next we have set three port numbers:
-  port 3050 is the port number that the service itself will be listening on for requests that are coming from inside the cluster. 
- the target port is the container's port number that the service will forward traffic to.
- ... and The nodePort is the port number that all the kubeworker nodes will start listening on at the node OS level, for accepting requests coming from outside the kubecluster. 
- Finally we have The selector. This is a really important setting. that's becuase it's the mechanism that attaches this service to our httpd pod. basically it says only forward traffic to pods that have the label, app equal to apache_webserver. This eliminates the problem of pods changing ip addresss becuase service forwards traffic using the label&selectors system rather than static ip addresses. 


So let's go ahead and create this service. 

```
$ kubectl apply -f configs/svc-nodeport-httpd.yml
$ kubectl get services -o wide
```

As part of creating this service, kubernetes took this service's name, along with it's ip address, and used them to register a dns entry in Kubernetes DNS.

We can test this by performing an nslookup inside one of the pods.

```
$ kubectl exec -it pod-centos -- bash
nslookup svc-nodeport-httpd
```

Ok it looks like nslookup isn't installed, so let's install it. y


```
yum whatprovides */nslookup
yum install -y bind-utils
```

Now let's try nslookup again

```
nslookup svc-nodeport-httpd
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   svc-nodeport-httpd.default.svc.cluster.local
Address: 10.105.186.109
```

This shows that we have a dns entry, which is the fqdn of our service along with it's ip address. Should this ip address change in the future then kubernetes DNS will get updated with the new ip address in real time. 



Now let's try out our new dns entry:


```
[root@pod-centos /]# curl http://svc-nodeport-httpd.default.svc.cluster.local:3050
<html><body><h1>It works!</h1></body></html>
```

As you can see we've now managed to access our apache pod via the service object, using the service's dns name rather than the pod's ip address.


We had to use port 3050 here because in the service spec we said that this service can only accept internal traffic on this port.

The 'default' in the fqdn actually refers to the namespace the service was created in. 

Also Since our centOS pod resides in the same namespace as the service, we can actually leave out the fqdn's basename, and just curl the service's name and it will still work:

```
$ curl http://svc-nodeport-httpd:3050
<html><body><h1>It works!</h1></body></html>
```

That worked because the resolv.conf has a default basename that the resolver use as a fallback if you don't specify the fqdn. 


```
$ cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Now, there's one final thing I wanted to show you before I end this video, and that's to do with the nameserver. I mentioned earlier that Kubernetes DNS is essentially built on top of CoreDNS. And this nameserver ip address leads to coredns. To show what I mean, let's exit out of the pod and then perform a search for this ip address. Here we find that it belongs to service called kube-dns:

```
$ kubectl get all -o wide --all-namespaces | grep 10.96.0.10
kube-system            service/kube-dns                    ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   4h34m   k8s-app=kube-dns
```

And if we look at what pods this service forwards traffic too, we find the coredns pods. 


```
$ kubectl get pods --namespace=kube-system -l k8s-app=kube-dns 
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-5wm6q   1/1     Running   0          4h45m
coredns-5644d7b6d9-kzpjj   1/1     Running   0          4h45m
```

These coredns pods that are responsible for maintaining all the DNS entries and providing the DNS lookup service. If coredns receives a nslookup request for a dns entry it has no knowledge of, such as doing an nslookup for google.com, then coredns will forward that request on to one of the internet's public dns servers, and then feedback that response back. 

That's it for this video. I'll see you in the next one. 

