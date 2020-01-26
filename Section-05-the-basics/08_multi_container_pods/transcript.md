# Transcript

Hello everyone, and welcome back. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

So far we've only explored single container pods. If you want, you can also build multi-container pods, and it's actually quite easy to set up. So in this video, I'm going to show what a 2 container pod looks like. Here's the yaml file that I'll use for this demo. 

```bash
tree configs/
code configs/pod-with-2-cntrs.yml
switch to bash terminal (ctrl+~) 
```

I've actually put together  this definition by copying and pasting yaml extracts from previous examples, so some of this might already look familiar to you. The first container is our hello-world apache container, and the second container is a standard cent-OS container. So let's see what we end up with when we create this pod. 

```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

Here we can see that our pod has 2 out of 2 containers ready. So that's all there is to it, we have created our first multi-container pod. 

There's a couple of interesting things you can do with  multi-container pods. First of all, the containers can interact with eachother via localhost. To see what I mean, lets first open up a terminal inside the cent-OS container:

```bash
$ kubectl exec pod-multi-cntr -c cntr-centos -it /bin/bash
```

Now let's trying curling localhost just to see what happens:

```bash
curl http://127.0.0.1
```

It's likely that you expected that this curl command to hang for a while and then timeout, since this is just a standard cent-OS container and doesn't have a web service running inside it. However what we can see here, is that not only did we get a successful response, but the response actually came from the other container in the pod, which is running apache. So you might be thinking,      what on earth is going on!


Ok, the way networking works inside a pod, is similar to how networking works inside a linux machine. All Linux machines have a special virtual network interface called the loopback interface. This loopback interface is used to route internal traffic between various processes on the Linux machine. The loopback interface has the ip address of 127.0.0.1, which is what processes use to forward traffic internally.  


For example, let's say you have a linux machine that has the  apache daemon running on it. Now if you open up bash terminal inside this machine and then curl 127.0.0.1, then behind the scenes the curl command starts up a process, and this process in turns forwards the curl request to the machine's loopback interface, the loopback interface then forwards the request on to the apache daemon's underlying process. The apache process then provides a response which gets sent back to the curl process. The curl process then feeds the response back to the curl command which then displays it on the bash terminal's standard output. 


In Kubernetes, the same kind of thing is going on. Where instead of a VM with a loopback interface, we have a pod with a loopback interface, and instead of processes talking to each other, we have containers talking to each other. 

So hopefully that now clears up what happened in this demo when we curled the localhost. Ok I don't need this pod any more so I'm going to delete it before moving on. 

```bash
kubectl delete pod ...
```



Another thing you can do with multi-container pods is that you can share folders between your containers using emptyDir volumes. Here's a yaml file to demo this:

```bash
tree configs/
code configs/pod-folder-share.yml
switch to bash terminal (ctrl+~) 
```

This is similar to the previous yaml file, but this time we have some volume definitions as well. Notice here that both containers are mounting the same volume. That's what makes folder sharing possible. 

Let's go ahead and create this. 


```bash
$ kubectl apply -f configs/pod-with-2-cntrs.yml
pod/pod-multi-cntr created
$ kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-multi-cntr   2/2     Running   0          30s   172.17.0.10   minikube   <none>           <none>
```

In this pod, the cent-OS container is appending new lines to the apache container's default homepage, index.html, thanks to the echo command in the infinite while-loop. This means that the apache container will end up displaying the new content as and when it becomes available. We can see this in action by curling our apache container from another pod. So let's try this out by first creating a new test pod, and then open up a bash terminal inside it:


```bash
kubectl run testpod --rm --image=centos --restart=Never -it --command -- /bin/bash
```

Ok, Now let's try curling our apache container's ip address:

```bash
kubectl exec -it -- curl http://xxxxxxx
```

We'll run this command a few times so that you can see the new content showing up:

```bash
kubectl exec -it -- curl http://xxxxxxx
kubectl exec -it -- curl http://xxxxxxx
```

As you can see, new lines are being added every few seconds. 

You can use other volumes types to share folders between containers, but using emptyDir volumes like this is quite a common use case. Especially if you want to restrict folder sharing to only containers in the same pod. And also if the data that's being shared, is no longer needed after the pod dies, then using emptyDir volumes is a great  way to clean up after yourself.



One thing to keep in mind when working with multicontainer pods, is that some commands will start asking for the container's name. For example the logs command:

```bash
$ kubectl logs pod-folder-share
Error from server (BadRequest): a container name must be specified for pod pod-folder-share, choose one of: [cntr-httpd cntr-centos]
```

Here the logs command doesn't know which container's logs you want to view. So you have use the -c flag to specify this  . 

```bash
$ kubectl logs pod-folder-share -c cntr-httpd
```


Other commands, such as the exec command will still work without having to explicitly specify the container's name. That's becuase they end up defaulting to the first container defined in the yaml definition, which in this example would be the apache container.  

```bash
$ kubectl exec pod-folder-share -- pwd
Defaulting container name to cntr-httpd.
Use 'kubectl describe pod/pod-folder-share -n default' to see all of the containers in this pod.
/usr/local/apache2
```

As you can see, the exec command even tells you which container it's defaulting too. However you can use the -c flag here as well, for example here's     how to run exec against the cent-os container.  

```bash
$ kubectl exec pod-folder-share -c cntr-centos -- pwd
```

This time the exec command has all the info it needs to know where exactly to run this command. 


Ok let's now move onto talking about.... when are good times to use    multi-container pods. As a general rule of thumb you should always stick to using single-container pods, however there are some use cases where you have to use multicontainer pods, such as when utilizing a service mesh technology like Istio.

So if you do have to use multicontainer pods, you need to ensure that your pod is made up of only one main container, and the rest are helper containers. The main container is the container that provides the core application that the pod was built for. Whereas the helper containers provides a supporting role to the main container. Also in the yaml definitions, the main container must be listed first, followed by all the other helper containers. So in this example the main container is apache, and centos is the helper container.

When researching on the internet, you'll find that the helper containers are often referred to by other names, depending on the particular role that it performs, they are:   

- sidecar containers
- ambassadar containers
- and Adapter containers  

Check out the study guide to learn more about what these are. 

That's it for this video, See you in the next one. 

