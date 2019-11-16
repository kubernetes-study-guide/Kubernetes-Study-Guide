# Transcript

TODO: ....

Structure:
vscode
-> powerpoint internalpodcommunication vs pod-to-pod communication
-> vscode


## powerpoint slide

Hello everyone one, and welcome back.


In the last video we saw how containers that lives inside the same pod can talk to each other using the loopback interface, 127.0.0.1. 

Another common use case is to have pods talking to each other, in other words, pod-to-pod communication? One way to achieve that, is by using using ip addresses. Each pod gets it's own private IP address auto-assigned to it at the time of it's creation, and we can use these IP addresses to get one pod to send requests to another pod. 

## vscode

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

Here we can see that Kubernetes has automatically assigned a private IP address to each pod. Let's now try execcing into the centos pod and then send a curl request to the apache pod's ip address:


```bash
kubectl exec ... -- curl http://xxxxxx
```

Cool, that worked. Our centos pod managed to send a request to the apache pod and got a successfuly response. 

However, it's actually bad practice to use ip addresses. There are a few reasons for this. But the main reason is that there's no gaurantee that a pod will always have the same ip address. If for whatever reason Kubernetes needs to delete and recreate a pod, then there's a chance that the new pod ends up with a different ip address. 

I only used IP addresses in this demo, just to give you an idea about how kubernetes networking works behind the scene.

Another reason to avoid using IP addresses, is that they're not meaningful or easy to keep track of. Let's take phoning a friend using your smart phone as an analogy. When you make a phone call to your friend, you do that by selecting their name from your phone's address book,you don't manually dial their number. Using IP address is a bit like manually dialing a number. Using IP addresses is a bit like manually dialling a number. 

That's why it would be more convenient if we could use names such as http://apache.localcluster rather than a random looking ip adderss. 

```popup
curl http://apache.localcluster
```


That's actually possible thanks to Kubernetes DNS along with creating Service Objects. We'll cover how that's done in the next couple of videos. 