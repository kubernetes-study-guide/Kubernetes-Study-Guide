# Transcript

TODO: ....

Structure:
vscode
-> swipe to httpd docker hub home page


## vscode


Earlier we looked at how to view a container's main process. As a reminder let me quickly do that again, using our hello world pod. 


```bash
tree configs/
code code configs/pod-httpd.yml
switch to bash terminal (ctrl+~) 
kubectl apply -f configs/pod-httpd.yml
kubectl get pods
kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
apt-get update
apt-get install -y procps
ps -ef
```

As you can see here, we have the primary process as identified by the process id of 1. and on the right we can see the command that was executed to start that process. 


But where did this startup command come from? In otherwords, how did the container know that it needed to execute this particular command in order to start the primary process? The answer is that this startup command is baked into the image itself by the dockerfile. In the dockerfile, the startup command is specifed using either the CMD or ENTTRYPOINT settings, or a combination of both. 


# website - https://hub.docker.com/_/httpd - swipe right. 

For example here's the httpd image's web page. Clicking on the dockerfile link takes us to github where we can view the dockerfile's content.

```web-tasks
click on link
```


At the bottom we find the CMD setting.


```web-tasks
scroll to bottom by dragging scroll bar
```


 This CMD command actually runs a shell script which was copied into the image. So let's go up one level and then take a look at this shell script.


```web-tasks
scroll to top by dragging scrollbar
click up one level
```

Here we can see that actual command that starts up the primary process. 

## vscode

This matches up with what we saw in our ps output. 


Since this container is designed to provide an ongoing web service, it means that this baked-in command starts a continuously running process.


However there are other images where the baked-in command only starts-up a shortlived process. For example the official centos image is a general purpose image that only comes with a shortlived bash command baked-in. 


So if you create a pod with this image, then the container will just keep dying repeatedly. Let's demo this with the following yaml file:

```bash
tree configs/
code configs/pod-centos-shortlived.yml 
```

Here we have a basic single container pod. By the way I've specified the image's tag here, just to show how to use a particular image version. if you leave this out, like we did with our hello-world pod then, then it defaults to using the tag called latest. 


Ok let's apply this now. 


```bash
kubectl apply -f configs/pod-centos-shortlived.yml
```

Now lets run get pods command a few times to see the progress:

```bash
$ kubectl get pods -o wide
$ kubectl get pods -o wide
$ kubectl get pods -o wide
$ kubectl get pods -o wide
```

Here we can see is that as expected the container keeps dying and this pod keeps building new replacement containers. As a result the restart number is creeping up over time.

Now in Kubernetes we can override the baked-in command, as shown in this yaml file:


```bash
tree configs/
code config
```

Here we have two settings, command is the kubernetes equivalent of dockerfiles Entrypoint setting. And similarly 'args' is equivalent for the dockerfile's CMD setting. These 2 settings will override the baked in start-up command. 

Here we're saying that we want to feed in this multiline string value into bash. We also used the -c flag to tell bash to treat this multiline string as a command. 

This effectively ends up running a shell script. Where the shell script is an infinitely running while-loop that prints out the date every 5 seconds. 

To better understand how this works, let's simulate how this shell script would run if ran it directly on my macbook. 



```bash - do some copy and paste. 
$ /bin/bash -c '
>         while true ; do
>           date 
>           sleep 5 
>         done'
```

Here we used single quotes to feed in the multiline string as a single arguement to the -c flag. Now let's run this. 


```instruction
press enter
```

As you can see, the script is now printing out the date every 5 seconds. ok let's cntrl+c out of the script. Now let's create this pod:

```bash
$ kubectl apply -f configs/pod-centos-ongoing.yml
kubectl get pods
```

It looks like that has now worked. let's take a look at what the resulting process looks like:

```bash
kubectl exec pod-centos -c cntr-centos -- ps -ef
```

Let's take a look at the logs to see if the timestamp is being sent to the standard output:

```bash
$ kubectl logs -f pod-centos -c cntr-centos
```

Notice I've used the -f flag here, that's so that we can follow the logs. Everything looks good here so lets cntrl+c out of that. 

you can also attach your terminal directly to the main process's standard output. That's done using the attach command:

```bash
$ kubectl attach pod-centos -c cntr-centos
```

Here we can again see the date being echoed out every 5 seconds. 


Now, going back to the yaml file, notice the how I wrote the command setting. This is actually yaml syntax for writing a list as a single line. There are other ways to write this yaml file to achieve the same result. I'ved included them in the more-sample folder. 

Ok, that's it for this video. See you in the next one. 
























You can still use these images in your pods, 

So to get this work, we need to 



Some images doesn't come with a baked-in command at all. in which case this would sets what the startup command should. 



, such as the official centos image, that comes with process 


 are general purpose images, and these  image's dockerfil






The whole point of having containers is to run a Linux process inside them. That process in turn provides a service that is of value to us. You can find what that process is by taking a look inside a pod. For example let's create our hello-world pod again

```bash
tree configs/
code 
```




But what start's that process in the first place? The answer is, commands.




The main reason we use containers is so to run a process that provides a service that's of value to us. This process can be in the form of a command or a shell script. 

if a image doesn't come with a predefined command baked into it, then you need to define an ongoing command in the pod definition instead. 