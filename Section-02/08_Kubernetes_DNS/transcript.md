In the previous video we saw how containers in the same pod can communicate with each other using the localhost. Which is fine, since the loopback's interfaces ip address of 127.0.0.1 is a standard and never changes.

However when it comes to pod-to-pod communications, we can do that using pod ip addresses. To demo this, I'll need to create 2 pods, after that I'll run a curl command from one pod to access the service exposes by the other pod. So for this demo I'll create an apache pod cent-os pod. I'll then run a curl command from the centos pod to reach the apache pod. here are the 2 yamls files I'll use to create these pods:

```bash
code ... 
```

These are actually copies of yaml files I've used in earlier demos. Now let go ahead and create these pods.  



```bash
$ kubectl apply -f configs/pod-httpd.yml -f configs/pod-centos.yml
pod/pod-httpd unchanged
pod/pod-centos unchanged
kubectl get pods -o wide
```

Here we can see what our apache pod's ip address. This is the ip address I need to curl to from the cento-os pod. So let's try that now:


```bash
kubectl exec ... -- curl http://xxxxxx
```

Here we can see that we've managed to get one pod to communicate with another pod using ip addresses. However direclty using pod ip addresses is a bad idea for a numbers, one of the main reason being that pod ip numbers can change over time if and and when the pod dies and get's recreated. That's where service objects comes into the picture. With service object you can access a pod via the service object,rather than using the pod's ip address. Let's demo this by creating a nodeport service, here's the yaml file I'll use to create nodeport service:

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

He we can see that our service object forwarded our curl request to the apache pod, which then sent the response. That's because this service and apache pod are associated with each other through label and selectors, as covered in earlier videos. One way to confirm this association is by using the endpoints command:

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