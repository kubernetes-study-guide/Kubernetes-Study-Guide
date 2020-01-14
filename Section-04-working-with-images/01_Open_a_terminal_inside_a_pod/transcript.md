A common thing a lot of people do when using a particular image for the first time, is to take a look inside the image. That's done by spinning up container from that image and then attach your terminal to that container. This is something I often do with docker using the Docker run command. 


However you can also do the same kind of thing in Kubernetes, using the kubectl run command. In fact the docker run command and the equivalnet kubectl run command looks quite similar. That's why for this demo I'm going to show you both of these techniques side-by-side.


Before I can start my demo, I need to a bit of prepartion work. First I'll open up a new terminal window 


```
gui action
```


and I'll break this window down into 4 panes. 


```
gui action
```

I'll use the left to panes for the docker demo, and the right two panes will be used for the equivalent kubernetes demo. 


Next I'll use the top left pane to list out my running docker containers. 

```
watch -n 1 docker container ls --all
```


Notice I'm using the 'watch' command here. This utility is used for running the same command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh internal to one second, that's to override the default, which is every 2 seconds.  This is a simple technique I like to use to monitor what's going on in near realtime. At the moment we don't have any containers. At the moment we don't have any containers. 


While I'm at it, I'll also use the top right pane to get a list of pods:

```
watch -n 1 kubectl get pods
```

At the moment we don't have any pods. 

Ok that's all the preparation out of the way. Now we can start the actual demo. Let's first take a look at the docker way of creating a bash terminal inside a container. For example heres the docker run command to open up a bash terminal to look inside the official centOS image:

```
$ docker run -it --rm --name client centos bash
cat /etc/redhat-release
```

Here I've created a new contianer which I've called 'client'. I've used the official centOS image to create the container. 


Now there's a kubernetes equivalent to this docker run command, and that's the kubectl run command. Let's take a look at what that looks like:

```
kubectl run --rm=true -it client --image=centos --restart=Never -- bash
```

Here I'm telling kubectl to spin up a container from the centos image and run the command that comes after the double-dash, which in this case is just bash. I've asked kubectl to give this pod the name "client". I've also requested that my current terminal should be attached to this bash session as an interactive terminal. The rm flag tells kubernetes to delete this pod after the bash process ends, which will happen when I exit out of the container. The --restart=Never setting tells kubernetes to not restart the pod if it stops running. Don't worry too much if this command doesn't make a lot of sense at the moment. I'll explain this command in more detail when we come talk about Kubernetes deployments in the course. As you can see the docker run and kubectl run commands look quite similar. The reason I'm highlighting the similarity is becuase if you're coming from a docker background, then these similarities makes using kubectl just that little bit easier to get to grips with. 


Let's hit enter now. 

```
action
```

As expected we now have a pod called client that has a single container running inside it, and our terminal is now attached to that container's bash session. We can now go ahead and run commands inside that container. 

```
pwd
ls -l
```


I'll also be using this technique quite a lot to spin up temporary pods which I'll then use for testing other pods. You'll see what I mean as we go through the course. Also from here on out, whenever I create one of these test pods, I'll always name them 'client' just to keep things consistent in this course. 


Ok let's exit out of our containers and and pods and clear the screen. 



Now what if you want to create a bash session inside an existing container?




For this demo, I'll create a docker container using the httpd image, and I'll name this container, webserver:

```
docker run --detach --name webserver httpd
```

Ok now I have a running container. 


Also here's the equivalent command for creating pod with a httpd container running inside it:

```
kubectl run webserver --image=httpd --restart=Never 
```

Ok we now have a running pod. Notice how similar these 2 commands are again. 



Now in the workd of docker, to get inside a container, you need to use the docker exec command. 

```
$ docker exec -it webserver bash
pwd
```

This creates a bash session inside this container and attaches my terminal to that bash session.


Now when it comes to Kubernetes, I can take a similar approach, using the kubectl exec command. 

```
$ kubectl exec -it webserver -- bash
```

Once again notice how similar these 2 commands are. 

