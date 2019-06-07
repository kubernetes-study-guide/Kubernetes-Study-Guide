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

Now as I said earlier, the whole point of containers is to run a process inside them. And that process in turn provides a service that is of value to us. From our earlier videos we already that this container provides a web service. But what is the underlying process that provides that service? We can find that out by using the ps command:

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

In containers, the process with the process id of 1 is the container's primary process. On the right we can see the command that was executed to start this process. We'll cover more about these commands in a later video. The main process also spawned a few child processes. As indicated by the fact that these processes have parent process id of 1. we can see this parent/child relationship a bit more visually using pstree:

```bash
pstree -an
```

Here we can see there are few child processes. 


A container's primary purpose is to run the main process. If that process comes to an end then the container will shutdown. So what happens if one of the pod's container dies? Well we can try it out and see what happens. 

Lets first exit out of the container. 

```bash
exit
```

Now let's run get pods again, just as a reminder. 

```bash
kubecelt get pods -o wide
```

Now let's simulatte a container failure by running the kill command:

```bash
kubectl exec pod-httpd -c cntr-httpd -- kill 1
```

This should have killed a the container. Now lets do get pods again:


```bash
kubectl get pods -o wide
```

Notice that this time there has been a restart. This restart doesn't mean the pod has been restarted since the pod's age is about the same as before. This restart means that the pod has restarted one of it's containers. Basically pods are self healing, so if a container shuts down then the pod will spin up a new container to replace it. You can confirm that this is what really happenedy by taking a look at the container's logs:

```bash
$ kubectl logs pod-httpd -c cntr-httpd
```

When using the logs command you should specify the container's name, since a pod can have multiple containers. You also need to specify the pod's name.

This shows the logs for the current container, which replaced the previous container. The timestamps indicated that this container has started up only a couple of minutes ago. We can also view the previous containers log using the previous flag:

```bash
$ kubectl logs pod-httpd cntr-httpd --previous
```

Here we can see the exact time the container received the kill signal. This timestamp is just before our new container's timestamp, which correlates with what we expected. 


If you have containers that keep unexpectedly restarting then these log commands are a great way to troubleshoot what's going on. 


There's one more thing I wanted to point out, pods are designed to only host containers that have a continuously running main process. In other words the containers in your pod needs to be continuously running. 

However there can be containers that have a main process that is only supposed to perform a specific task and then end naturally, which in turn would shut down it's container as well. These types of short-lived containers shouldn't be placed inside pods. Otherwise the pod will view these shutdowns as something unexpected and will keep spinning up replacement containers. 


Instead you should run these shortlived containers inside a job or cronjob objects. We'll cover these later. 

That's it for this video. See you in the next one. 






 if our container's process ends naturally, becuase the process finished whatever it was supposed to do, rather than us killing it with the kill command.