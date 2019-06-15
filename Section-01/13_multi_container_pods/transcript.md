# Transcript

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

In this video we're going to build a multicontainer pod. In our demo, we'll build a pod that has 2 containers running inside it. Here's the yaml file we'll use to create this. 

```bash
tree configs/
code configs/pod-httpd.yml
switch to bash terminal (ctrl+~) 
```

I've actually created this definition using yaml extracts from previous examples. The first container is a hello-world apache webserver container, and the second container is a standard centos container with an infinite while loop, which keeps the cento container running continuously. So let's see what we end up with when we create this pod. 

```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
$ kubectl get pod -o wide
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

Here we can see that our pod contains 2 containers. That's indicated by the fact that our pod has 2 containers ready out of total of 2 containers. Containers inside the pod behaves a little like processes inside the same virtual machine, in the sense that the containers can interact with each other using localhost networking namespace. Let's see this in action by starting up a terminal inside the centos container:

```bash
$ kubectl exec pod-multi-cntr -c cntr-centos -it bash
```

Now this container doesn't have apache running on it. However if I curl the localhost I get the following:


```bash
curl http://localhost
```

As you can see here we got a successful response. That's becausee all containers inside a pod are part of the same network namespace. That means that when we run the curl command from our centOS container, our web container ended up responding to that request. You can think of it as all the containers in a pod have the same loopback interface attached to them. You can also share folders between containers in a pod. We'll cover how to do that in a later video. 



In Kubernetes, the first container that's listed in your yaml file, is treated to as the main container, and all the other containers are treated as supporting containers. 





commands where you need to specify a container name, e.g. logs commands, if you omit the -c flag, then the command will default to using the main container's log. 