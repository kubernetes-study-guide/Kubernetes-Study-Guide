A common thing a lot of people do when using a particular image for the first time, is to take a look inside the image by spinning up container from that image and then attach a bash terminal to that container. Now you most likely know how to do that using docker. 

For example if you want to Let's first take a look at the docker way of doing this. For this example, I'm goint to create a shell session inside the official centos image, and that's done using the docker run command:

For example if you want to open up a bash terminal to look inside the official centOS image, then you can do that using docker run:

```
$ docker run -it --rm --name client centos bash
cat /etc/redhat-release
```

Here I've created a new contianer which I've called 'client'. I've used the official centOS image to create the container. 

Ok I'll exit out of that now

```
exit
```

Now there's a kubernetes equivalent to this docker run command, and that's the kubectl run command. Let's take a look at what that looks like:

Now kubernetes let's you do a similar thing using kubectl run. Here's what the equivalent command looks like:

```
kubectl run --rm=true -it client --image=centos --restart=Never -- bash
```

Here I'm telling kubectl to spin up a container from the centos image and run the command that comes after the double-dash, which in this case is just bash. I've asked kubectl to give this pod the name "client". I've also requested that my current terminal should be attached to this bash session as an interactive terminal. The rm flag tells kubernetes to delete this pod after the bash process ends, which will happen when I exit out of the container. The --restart=Never setting tells kubernetes to not restart the pod if it stops running. Don't worry too much if this command doesn't make a lot of sense at the moment. I'll explain this command in more detail when we come talk about Kubernetes deployments in the course. 


Ok before I run this command let's first get a list of all our pods. I'll do that in a another terminal:

```
# open another terminal
watch -n1 kubectl get pods
```

Notice I'm using the 'watch' command here. This utility is used for running a command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh internal to one second.  This is a simple technique I like to use to monitor what's going on in near realtime. Ok I'll hit enter to start the monitoring. 

```
<enter>
```

At the moment we don't have any pods. Ok let's now hit return on the kubectl run command:


```
<enter>
```

As expected we now have a pod called client that has a single container running inside it, and our terminal is now attached to that container's bash session. We can now go ahead and run commands inside that container. 

```
pwd
ls -l
```



I'll be using this command a lot for spinning up temporary pods which I'll then use for test other pods. You'll see what I mean as we go through the course. 





















Notice I'm using the 'watch' command here. This utility is used for running a command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh internal to one second.  This is a simple technique I like to use to monitor what's going on in near realtime. Ok I'll hit enter to get that started. 

```
<enter>
```

At the moment we don't have any containers. Ok let's now hit return on the docker run command:

```
$ docker run -it --rm --name client centos bash <enter>
```

So far so good, it looks like we now have a running container and our terminal is attached to that container's bash session. 

```
cat /etc/centos-release
```

Ok let's exit out of the container now. 

```
exit
```

That caused the container's bash session to end, and as soon as that happened our container got deleted. 


Now let's look at how to do the same thing in kubernetes. Here's what the corresponding kubectl command looks like. 



```

```







Ok what if you have an existing container that you want to look inside. Ok let's first show how that's done in docker world before showing the kubernetes way. 







In the quiz section, there are several exercises you can try to create,

using the following types of apps:


mysql
jenkins
wordpress
