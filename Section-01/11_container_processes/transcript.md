# Transcript

TODO: ....

Structure:
vscode
-> vscode
-> github: 

## vscode

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

The whole point of having containers is to run a Linux process inside them. That process in turn provides a service that is of value to us. You can find what that process is by taking a look inside a pod's container. For example let's create our hello-world pod again by using the following yaml file:



```bash
tree configs/
code code configs/pod-httpd.yml
switch to bash terminal (ctrl+~) 
```

This is the same hello world pod from earlier. So let's create it:

```bash
$ kubectl apply -f configs/pod-httpd.yml
kubectl get pods
```

Now let's start a bash session inside the container:

```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
```

Now as I said earlier, the whole point of containers is to run a process inside them. And that process in turn provides a service that is of value to us. From our earlier videos we already know that this container provides a web service. But what is the underlying process that provides that service? We can find that out by using the ps command:

```bash
ps -ef
apt -v
```

It looks like ps isn't installed in our container. But I do have apt available so I'll use that to install ps:

```bash
apt-get update
apt-get install -y procps

```

Ok now let's try again:

```bash
ps -ef
```

Ok that has worked this time. In containers, the primary process has a process id of 1 so this is our primary process. On the right we can see the command that was executed to start this process. We'll cover more about these commands in a later video. The primary process has also spawned a few child processes. That's indicated by the fact that these processes have parent process id of 1. we can see this parent/child relationship a bit more visually using pstree:

```bash
pstree -an
```

Here we can see there are few child processes. 


A container's main purpose is to run this primary process. If that process comes to an end then the container will have served it's purpose and will then shutdown. So what happens if one of the pod's container dies? Well, we can try this out and see what happens. 

Lets first exit out of the container. 

```bash
exit
```

Now let's run get pods again, just as a reminder. 

```bash
kubectl get pods -o wide
```

Now let's prematurely end the primary process by sending it a kill signal:

```bash
kubectl exec pod-httpd -c cntr-httpd -- kill 1
```

This should have killed the container. Now lets do a get pods again:


```bash
kubectl get pods -o wide
```

Notice that this time there has been a restart. This restart doesn't mean the pod has been restarted since the pod's age is about the same age as before. This restart means that the pod has restarted one of it's containers. Basically pods are self healing, so if a container shuts down then the pod will spin up a new container to replace it. We can confirm that this is what really happenedy by taking a look at the container's logs:

```bash
$ kubectl logs pod-httpd -c cntr-httpd
```

The logs command displays the standard output, and error output, for the primary process. 

When using the logs command you should specify the container's name, since a pod can have multiple containers. You also need to specify the pod's name.

Here we can the logs for the current container, which replaced the previous container. The timestamps indicated that this container has started up only a couple of minutes ago. We can also view the previous containers log using the previous flag:

```bash
$ kubectl logs pod-httpd cntr-httpd --previous
```

Here we can see the exact time the container received the kill signal. This timestamp is just before our new container's timestamp, which correlates with what we expected. 


If you ever have containers that keep unexpectedly restarting, then viewing the logs like this, is a useful way to investigate what's going on. 


There's one more thing I wanted to point out, pods are designed to only host containers that have a continuously running primary process. In other words, the containers in your pod needs to be continuously running. 

However there can be containers that have a main process that is only supposed to perform a specific task and then end naturally, which in turn would cause the container to shut down too. These types of short-lived containers shouldn't be run inside pods. Otherwise the pod will view these shutdowns as something unexpected and will keep spinning up replacement containers. 


Instead you should run these shortlived containers inside other object types, such as a job or cronjob object. We'll cover these later. 

That's it for this video. See you in the next one. 