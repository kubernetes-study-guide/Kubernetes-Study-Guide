# Transcript

TODO: ....

Structure:
vscode
-> swipe to httpd docker hub home page


## vscode

Hello everyone one, and welcome back.

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
# adjust font size
git add . ; git commit -m wip ; git push
kubectl delete all --all --grace-period=2
clear
cmd+k
pwd
```


Earlier we looked at how to view a container's primary process. L et me quicky do that again using our hello world pod. 


```bash
tree configs/
code configs/pod-httpd.yml
switch to bash terminal (ctrl+~) 
kubectl apply -f configs/pod-httpd.yml
kubectl get pods
kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
apt-get update
apt-get install -y procps
ps -ef
```

As you can see here, we have the primary process as identified by the process id of 1. and on the right we can see the command that was executed to start that process. 


So my question is, where did this startup command come from? In otherwords, how did the container know that it needed to execute this particular command in order to start the primary process? The answer is, this startup command is baked into the image itself by the dockerfile. In the dockerfile, the startup command is defined using either the CMD setting, or the ENTTRYPOINT setting, or a combination of both. 


# website - https://hub.docker.com/_/httpd - swipe right. 

For example here's the docker hub page for the official httpd image    . Clicking on the dockerfile link takes us to github, where we can view the dockerfile that built this image.

```web-tasks
click on link
```


At the bottom we find the CMD setting.


```web-tasks
scroll to bottom by dragging scroll bar
```


      This setting, actually runs a shell script which was copied into the image in an earlier step. So let's go up one level and then take a look at this shell script.


```web-tasks
scroll to top by dragging scrollbar
click up one level
```

Here we can see that actual command that starts up the primary process. 

## vscode

This matches up with what we saw in our ps output. 

# TODO-start
Another way to find out what the start up command is, is by using docker

```bash
$ docker run -d httpd
$ docker container ls --all
```

Although it doesn't quite give the right answer here since this particular runs the root process using a shell script.
# TODO-end




However there are other images where the baked-in command only starts-up a shortlived process. For example, the official centos image is a general purpose image that only starts bash, which in turn runs for less than a second. 

# TODO-start
Another way to find out what the start up command is, is by using docker

```bash
$ docker run -d centos
$ docker container ls --all
```

# TODO-end


So if you create a pod with this image, then the container will just keep dying repeatedly. Let me show you what i mean with this yaml file:


```bash
tree configs/
code configs/pod-centos-shortlived.yml 
```

Here we have a basic single container pod, where I'm  using  the official cent-      OS image. 
Ok let's apply this now. 


```bash
kubectl apply -f configs/pod-centos-shortlived.yml
```

Now lets run the get pods command a few times to see the progress:

```bash
$ kubectl get pods -o wide
$ kubectl get pods -o wide
$ kubectl get pods -o wide
$ kubectl get pods -o wide
```

As you can see   the cent-OS container keeps dying and the pod keeps building new replacement containers. As a result the restart number,  just keeps going up.

Now in Kubernetes we can override the baked-in command, as shown in this yaml file:


```bash
tree configs/
code config
```

Here we have two settings, command is the kubernetes equivalent of the dockerfile'  s Entrypoint setting. And similarly 'args' is the equivalent for the dockerfile's CMD setting. These 2 settings will override the baked in start-up command        . By the way, some images might not come with a baked-in command at all. In which case you can use these 2 settings to just set the startup command.

Here we used the -c flag. this flag requires a string input. The -c flags then instructs bash to run this string as a command. In our case, this string is going to be a multiline string and that we're feeding into this    flag via the command setting.


This effectively ends up running a continously running shell script, whose only job is to print out the date every 5 seconds. This is a bit of a crude technique, but it is quite useful for testing purposes. 

To help you understand what this startup command looks like, let's try running it directly on my workstation. 



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

As you can see, the script is now printing out the date every 5 seconds. ok let's cntrl+c out of this script a go ahead with creating this pod:

```bash
$ kubectl apply -f configs/pod-centos-ongoing.yml
kubectl get pods
```

Ok it looks like that has worked   . Let's check that the pod is still running without any restarts:

```bash
kubectl get pods -o wide
```




So far so good. let's now take a look at what the resulting process looks like:

```bash
kubectl exec pod-centos -c cntr-centos -- ps -ef
```

This confirms that our shell script is now running continuously, as the primary process which is what we were hoping for   . Let's take a look at the logs to see if the timestamp is being sent to the standard output:

```bash
$ kubectl logs -f pod-centos -c cntr-centos
```

Notice I've used the -f flag here, that's so that we can follow the logs. Everything looks good here so lets cntrl+c out of that. 

you can also attach your terminal directly to the main process's standard output. That's done using the attach command:

```bash
$ kubectl attach pod-centos -c cntr-centos
```

H ere we can see the date is being echoed out every 5 seconds, like we saw earlier using the logs command. 


Now let's turn our attention back to this yaml file, one thing you may have noticed, is the wierd syntax I used to write the command setting. This is actually yaml syntax for writing list as a single line. 

#TODO - Start
Also for args, args can only take an list. 

```bash
$ kubectl explain --recursive pod.spec.containers.args
```

As indicated by the field definition. 

So in our case, we specified a single value list, where the pipe character is standard yaml syntax for defining a multiline string. 


#TODO - End

 However there are a few other ways to write this yaml file to achieve the same end result. I've included some examples of them in the more-samples folder, in case you want to take a look at them. 

```bash 
ls configs/more-samples
code ...    
```


Finally I should point out that these two settings are not the only ways to run custom commands inside pods. There are other options, such as initcontainers, lifecycle hooks, and healthcheck probes:

```popupwindows
- making using of initcontainers,
- or use postStart and preStop lifecycle hooks
- and there's also Liveness and Readiness Probes
```

We'll cover all these later in the course. 

Ok, that's it for this video. See you in the next one. 
