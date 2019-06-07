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
kubecelt get pods -o wide
```







Here the process that has the process id of 1, is the main process that this container has been created to run. 


But what start's that process in the first place? The answer is, commands.




The main reason we use containers is so to run a process that provides a service that's of value to us. This process can be in the form of a command or a shell script. 

if a image doesn't come with a predefined command baked into it, then you need to define an ongoing command in the pod definition instead. 