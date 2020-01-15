Hello everyone, and welcome. 

If there's a particular image you want to use that you'be never used before, then you might want to first check what files, folder's and software packages that image comes with. One way to do that is by reviewing that image's dockerfile. 

But the way I prefer to do it is by spinning up container from that image and then attach my terminal to that container. After that I can explore the image from the command line. You can do this with docker by running the "docker run".


However you can also do this using kubernetes, using the "kubectl run" run commnd. Both of these commands look quite similar, So for this demo I'm going to show you both techniques side-by-side, just so you can see the similarities. 

To do that I've opened up 4 terminals, and I'll used the 2 left terminals for the docker demo, and the 2 right terminals for the equivalent kubernetes demo. 



I'll start with listing out my running docker containers in the top left terminal. 

```
watch -n 1 docker container ls
```


Notice I'm using the 'watch' command here. This utility is used for running the same command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh interval to one second, otherwise the interval defaults to 2 seconds.  I like using the watch command because it's simple technique for monitoring what's going on in near realtime. At the moment we don't have any containers.


On the kubernetes side, I'll get a list of pods:

```
watch -n 1 kubectl get pods
```

As you can see, at the moment we don't have any pods either. 

Ok that's all the preparation out of the way. Let's now try that docker run command. For this example I'm going to use the docker run command to open up a bash terminal to look inside the official centOS image:

```
$ docker run -it --rm --name client centos bash
cat /etc/redhat-release
```

Here I've created a new container which I've called 'client' and I've used the centOS image to create this container. I've requested docker to run the bash command as the primary process inside this container, and also attach my terminal to this bash process as an interactive terminal. 


Now here's the equivalent kubectl run command:

```
kubectl run --rm=true -it client --image=centos --restart=Never -- bash
```

ok, Let's first go over this command before I hit return. 

Here I'm telling kubectl to spin up a container from the centos image and run the command that comes after the double-dash, which in this case is just bash. I've asked kubectl to give this pod the name "client". I've also requested that my current terminal should be attached to this bash session as an interactive terminal. The rm flag tells kubernetes to delete this pod after the bash process ends, which will happen when I exit out of the container. The --restart=Never setting tells kubernetes to not restart the pod if it stops running. Don't worry too much if this command doesn't make a lot of sense at the moment. I'll explain this command in more detail when we come talk about Kubernetes deployments in the course. As you can see the docker run and kubectl run commands look quite similar. The reason I'm highlighting the similarity is becuase if you're coming from a docker background, then these similarities makes using kubectl just that little bit easier to get to grips with. 

By the way notice how similar these 2 commands look. It's sort of a nice symmetry. 


ok Let's hit enter now. 

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

