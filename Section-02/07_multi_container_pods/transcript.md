# Transcript

Hello everyone, and welcome back. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

So far we've only explored single container pods. If you want, you can also build a multi-container pods, and it's actually quite easy to set up. So in this video, I'm going to show what a 2 container pod looks like. Here's the yaml file that I'll use for this demo. 

```bash
tree configs/
code configs/pod-with-2-cntrs.yml
switch to bash terminal (ctrl+~) 
```

I've actually created this definition by recycling yaml extracts from earlier examples. The first container is our hello-world webserver container, and the second container is a standard cent-OS container. So let's see what we end up with when we create this pod. 

```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

Here we can see that our pod has 2 containers ready out of total of 2 containers. Containers inside the same pod have a few interesting capabilities. To see what I mean, lets first open up a terminal inside the centos container:

```bash
$ kubectl exec pod-multi-cntr -c cntr-centos -it /bin/bash
```

The cent-OS container doesn't have apache running on it. However if I curl localhost then I get the following:


```bash
curl http://localhost
```

As you can see here, we got a successful response. This response actually came from the web container. That happened because all containers inside a pod are part of the same network namespace. That means that when we run the curl command from our centOS container, our web container ended up responding to that request. So what on earth is going on here!

Ok the way networking inside a pod, is similar to how networking works on a linux machine. All Linux machines have special virtual network interface called the loopback interface. This loopback interface is used to route internal traffic between various processes on the Linux machine. The loopback interface has the ip address of 127.0.0.1, which is what internal processes use to forward traffic internally.  


For example, let's say you have a Ubuntu machine that has apache werbserver running on it. Now if you open up bash terminal inside this ubuntu machine and then curl 127.0.0.1, then behind the scenes the curl command starts up a process, this process in turns forwards the curl request to the machine's loopback network interface, the loopback interface then forwards the request on to the apache webservice's underlying process. This process then provides a response which gets sent back to the curl process. 


In Kubernetes, the same kind of thing is going on. Where instead of a VM with a loopback interface,we have a pod with a loopback interface, and instead of processes talking to each other, we have containers talking to each other. 

So hopefully that now clears up what happened when we run the curl command form the cent-OS container. 

Another cool thing you can do is that you can share a folder between your containers using emptyDir. Here's a ymal file to demo this:

```bash
tree configs/
code configs/pod-folder-share.yml
switch to bash terminal (ctrl+~) 
```

This is similar to the previous yaml file, but this time we have some volume defintions as well. Notice here that both containers are mounting the same volume. Let's go ahead and create this. 


```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

The cool thing here is that the cent-OS pod is generating new web content that the apache container then displays in realtime. We can see this in action by running the curl command from our first pod:


```bash
kubectl exec -it -- curl http://xxxxxxx
kubectl exec -it -- curl http://xxxxxxx
kubectl exec -it -- curl http://xxxxxxx
```

As you can see, new lines are being added every few seconds. 

You can use other volumes types to achieve the same effect, but using emptyDir volumes like this is quite a common use case.



One thing to bear in mind when dealing with multicontainer pods, is that some commands will start asking for the container name. For example the logs command:

In this scenario you just have to use the -c flag to tell the command which container you're interested in. 


Other commands, such as xxx will still work, becuase they end up defaulting to the first container defined in the yaml definition, which in this example would be the apache container.  

```bash
xxxxx
```

Now let's talk about when you should use multi-container pods. As a general rule of thumb you should always stick to using single-container pods, and you if do need to build multicontainer pods, then try to only have one main container, and the rest as helper containers. The main container is container that provides the main application, without which the pod becomes useless. Whereas the helper containers provides a supporting role to the main container. In this example the main container is apache, and centos is the helper container. So if the main container dies, then the pod can no longer provide a web service, whereas the pod can still offer a web service if just the helper container dies. 

That's it for this video, See you in the next one. 

