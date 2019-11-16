Hello everyone one, and welcome back.

Earlier we saw how you can do pod-to-pod communication using pod IP addresses. However IP address can change over time, and are difficult to keep track of. That's why the recommended approach is to use DNS names instead of IP addresses. Most Kubernetes setups comes with a DNS servicee pre-installed, which is commonly known as Kubernetes DNS. Kubernetes DNS is actually built on top of another open source software called CoreDNS.












In Kubernetes, you can also create your own easy to remember dns names for your pods. That's done by creating Service objects. Let's demo kubernetes dns by creating a nodeport service. 

```





With service object you can access a pod via the service object, rather than using the pod's ip address. Let's demo this by creating a nodeport service, here's the yaml file I'll use to create nodeport service:

```bash
code ...
```

Once again I've taken this yaml file from an earlier demo, where we've set port xx for pod-to-pod communication. So let's now go ahead and create this service:


```bash
kubectl apply -f ...
```

As you can see, service objects comes with it's own ip address. So let's try curling the services ip address:

```bash
kubectl exec -- curl....
```

Here we can see that our service object forwarded our curl request to the apache pod, which then sent the response. That's because this service and apache pod are associated with each other through label and selectors, as covered in earlier videos. One way to confirm this association is by using the endpoints command:

```bash
$ kubectl get endpoints svc-nodeport-httpd
NAME                 ENDPOINTS       AGE
svc-nodeport-httpd   172.17.0.3:80   8s
```

Here we can see our service will forward traffic to the following ip address at this very moment in time. This info get's update in realtime. 

Now we no longer have to worry about the a pod's ip number changing, because if it does change the our service will automatically update update the endpoint with the new pod ip address. 




There's a lot more to services than just setting up dns and we'll explore them later. 



and a centos pod. 

now let's curl by id address. Now lets do the same thing but using the nodeport service objects, 

then lets curl the service ip address. 

So far so good. 

However there a few problem with use ip address:

- ip address can change
- ip address are not informative. a dns name google.com is informative. 
- dns names have a logical naming structure that you can specify in your pods at creation time.  

To fix this, we can use kubernetes dns. That's something that comes preinstalled by default in most installer options, such as minikube. 

create another new, shorter article after this, called : clusterip services. 
lhoikkjkopk';


cat





## reference

https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/