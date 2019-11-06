In the last video we saw how containers that lives inside the same pod can talk to each other using the loopback interface, 127.0.0.1. 

However, what about if want one pod to talk to another pod? Well one way to do that is by using the ip addresses that Kubernetes auto assigns to each pod.  


To demo this, I'll need to create a apache pod, and a centos pod. I'll then run a curl command from the centos pod to reach the apache pod. here are the 2 yamls files I'll use to create these pods:

```bash
code ... 
```

These are actually copies of yaml files I've used in earlier demos. Now lets go ahead and create the 2 pods.  



```bash
$ kubectl apply -f configs/pod-httpd.yml -f configs/pod-centos.yml
pod/pod-httpd unchanged
pod/pod-centos unchanged
kubectl get pods -o wide
```

Here we can see that Kubernetes has automatically assigned IP addresses to each pod. So let's now see if our centos pod can talk to the apache pod. Now let's try sending a curl request from our centos pod, to the apache pod.  

This is the ip address I need to curl to from the cento-os pod. So let's try that now:


```bash
kubectl exec ... -- curl http://xxxxxx
```

Ok that has worked. We've managed to successfully get our centos pod to talk to the apache pod. 

However using pod ip addresses like this is actually bad practice. For example, there's no gaurantee that a pod will always have the same ip address. If for whatever reason kubernetes has to delete and recreate the apache pod, then the pod could end up with a different ip address. Also ip addresses are not easy to remember or keep track of. 

In the real world, we use DNS instead of raw ip addresses. For example if you want to access the Google search, you don't do that by typing the google server's ip address into your browser, instead you use it's dns name, google.com. 

In Kubernetes, you can also create your own easy to remember dns names for your pods. That's done by createing Service objects. 

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
