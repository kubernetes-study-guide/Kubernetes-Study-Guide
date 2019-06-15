# Transcript

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

So far we've only demo pods that only had a single container running inside it. However you can multiple containers running inside a single pod. In this demo, I'm going to show what a 2 container pod looks like. Here's the yaml file I'll use to create this. 

```bash
tree configs/
code configs/pod-with-2-cntrs.yml
switch to bash terminal (ctrl+~) 
```

I've actually created this definition using yaml extracts from previous examples. The first container is a hello-world webserver container, and the second container is a standard centos container with an infinite while loop. So let's see what we end up with when we create this pod. 

```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
$ kubectl get pod -o wide
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

Here we can see that our pod contains 2 containers. That's indicated by the fact that our pod has 2 containers ready out of total of 2 containers. Containers inside the same pod have a few interesting capabilities. To see what I mean, lets first open up a terminal inside the centos container:

```bash
$ kubectl exec pod-multi-cntr -c cntr-centos -it /bin/bash
```

Now this container doesn't have apache running on it. However if I curl the localhost I get the following:


```bash
curl http://localhost
```

As you can see here we got a successful response. This response actually came from the web container. That's because all containers inside a pod are part of the same network namespace. That means that when we run the curl command from our centOS container, our web container ended up responding to that request. 

You can think of this in terms of a pod is a bit like a virtual machine, and the containers as processes inside that vm, in which case the containers can interact with each other using the localhost networking namespace. That means that the pod's network loopback interface is simultaneously attached to all containers in that pod.

You can also share folders between containers in a pod. But we'll cover how to do that in a later video.

There's another thing I wanted to to show you, let me first exit out of this pod. Now let me run the logs command:

```bash - write command but not hit enter

```

Now normally I would also add the -c flag along with the containers name, so that kubectl knows exactly which container's log I'm interested in. However I'm not going to bother with that and just hit return


In Kubernetes, the first container that's listed in your yaml file, is treated to as the main container, and all the other containers are treated as supporting containers. 

These containers are referred to by other names, such as sidecar containers, secondary containers,...etc. 


As these names implies. the supporting containers aren't supposed to provide a pod's primary service. Instead it provides additional features, such as log forwarding using filebeat. 





commands where you need to specify a container name, e.g. logs commands, if you omit the -c flag, then the command will default to using the main container's log. 