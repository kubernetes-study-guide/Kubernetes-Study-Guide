# Transcript

TODO: ....

Structure:
vscode
-> powerpoint internalpodcommunication vs pod-to-pod communication
-> vscode


## powerpoint slide

Hello everyone one, and welcome back.


In the last video we saw how containers that lives inside the same pod can talk to each other using the loopback interface, 127.0.0.1. 

However, what about pod-to-pod communications? Well one way to do that is by using the ip addresses that gets auto-assigned to each pod.  

## vscode slide

Let's take a look at how that's done. Here I've opened up a bash terminal inside this video's topic folder.

```bash
pwd
```

In this folder we have a couple of pod yaml definitions.

```bash
code configs/pod-httpd.yml 
switch to bash terminal (ctrl+~) 
code configs/pod-centos.yml 
switch to bash terminal (ctrl+~) 
clear
cmd+k

```

The apache container is going to listen for requests on port 80, this in turn means that the apache pod itself will be listening on that port 80 too. so we should be able to send a http curl request to that pod and get a successful response. So lets go ahead and create the 2 pods.  



```bash
$ kubectl apply -f configs/pod-httpd.yml -f configs/pod-centos.yml
pod/pod-httpd unchanged
pod/pod-centos unchanged
kubectl get pods -o wide
```

Here we can see that Kubernetes has automatically assigned an IP address to each pod. So let's now use curl to see if our centOS pod can reach the apache pod, by sending the request to the apache pod's ip address:


```bash
kubectl exec ... -- curl http://xxxxxx
```

Ok that has worked. We've managed to get our centos pod to successfully talk to the apache pod. 

However, it's actually bad practice to use ip addresses. That's becuase there's no gaurantee that a pod will always have the same ip address. If for whatever reason kubernetes has to delete and recreate the apache pod, then there's a chance that the new apache pod could end up with a different ip address. 

The only reason I used IP addresses in this demo, is just to give you a behind-the-scenes look at how kubernetes networking works.

Also ip addresses are harder to use because they're not easy to remember or meaningful. A good analogy is when phoning a friend using a smart phone. When you make a phone call to your friend, you do that by selecting their name from your phone's address book. That's a lot easier than manually dialling their number. 

That's why in Kubernetes, it would be so much easier if we could assign meaningful names to our pods, and then get our pods to talk to each other with those names. 


That's actually possible, thanks to Kubernetes DNS, and we'll cover that next. 


```

