Hello everyone, and welcome. 

If there's a particular image that you're interested in using for the first time, then you might want to first check what files and software packages that image comes with. One way to do that is by reviewing that image's dockerfile. 

However the way going to show you is how to look inside the image itself. We can do that by spinning up a container from that image and then attach a bash terminal to that container. After that we can explore the image from the command line. We can do all this with docker by using the "docker run" command.


However you can also do this with kubernetes, using the "kubectl run" command, whic actually looks quite similar to the docker run command. So for this demo I'm going to show you both techniques side-by-side, just so you can see the similarities. 

To do that I've opened up 4 terminals, and I'll use the 2 left terminals for the docker demo, and the 2 right terminals for the equivalent kubernetes demo. 



I'll start with listing out my running docker containers in the top left terminal. 

```
watch -n 1 docker container ls
```


Notice that I'm using the 'watch' command here. This utility is used for running the same command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh interval to one second. However the default is 2 seconds if I didn't use this flag. Ok let's run this now. 

```
enter
```

As you can see, At the moment we don't have any containers running.


Likewsie, On the kubernetes side, let's get a list of pods:

```
watch -n 1 kubectl get pods
```

Here we can see that we don't have any pods either. 

Ok that's all the preparation out of the way. Let's now demo the docker run command. Let's say I want to look inside the official centOS image, in that case here's the docker run command I'm going to use:

```
$ docker run -it --rm --name client centos bash
cat /etc/redhat-release
```

Here I've created a new container which I've called 'client' and I've used the centOS image to create this container. I've requested docker to run the bash command as the primary process inside this container, and also attach my current terminal to this bash process as an interactive terminal. That's why my command prompt now looks a little. 


Now here's the equivalent kubectl run command:

```
kubectl run --rm=true -it client --image=centos --restart=Never -- bash
```

ok, Let's first go over this command before I hit return. 

Here I'm telling kubectl to spin up a pod with a single container running inside it. This container needs to be created from the centos image, and it must run the command that comes after the double-dash, which in this case is just bash. I've asked kubectl to name this pod "client". I've also requested that my current terminal is attached to this bash session as an interactive terminal. The rm flag tells kubernetes to delete this pod after the bash process ends, which will happen when I exit out of the container. The --restart flag tells kubernetes to create a pod rather than a deployment object. I'll explain more about deployment objects later in the course, but for now just accept that you need to set the restart flag to false if you want the kubectl run command to just create a single pod. 

By the way, notice similarities between these 2 commands look. It's kind of symmetrical which is pretty cool. 


ok Let's now hit enter. 

```
action
```

As expected we now have a pod called client that has a single centOS container running inside it, and our terminal is now attached to that container's bash session. We know that because our command prompt has now changed. We can now go ahead and run commands inside this container. 

```
pwd
ls -l
```


I'll be using the "kubectl run" command quite a lot in this course for spinning up temporary pods just for performing some quick tests. But when I do that I'll always name those pods 'client'. I'm using this naming convention so that you can differentiat between the actual application pods I'm demoing, and which pods are just being used to test those application pods. 

Ok let's exit out of our containers and pods. 

```
exit
```

Notice that as soon as exit out the container the container got deleted. The same goes for the pod object as well.  


One final thing I wanted to show you is that you can use kubectl run to spin up a pod just long enough to run a single command and then that pod gets deleted again. 

```
kubectl run --rm=true -it client --image=centos --restart=Never -- echo hello
```

As yoy can see, a pod started just long enouggh to run this command. 

Ok we'll take a break here. In the next vidoe I'll demo how to create bash session inside an existing container 

