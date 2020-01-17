Hello everyone, and welcome. 


In the last video we saw how we can look inide a image by spawning a container from that image, and then open up a bash terminal inside it. I demoed how to do that using the docker way and the kubernetes way, side by side.  

Now what if you want to create a bash session inside a container that's already running? That's something you might want to do if a container is misbehaving and you want to investigate why. 

Well, in docker, you can do that using the docker exec command, and in Kubernetes you can do it using the kubectl exec command. Both of these commands once again look quite similar, so I'll demo them side-by-side like last time. 

Also like before, I'll start monitoring my list of containers and pods. 

```
watch -n 1 kubectl get pods
watch -n 1 docker container ls
```

As you can see we don't have any or pods running. 

For this demo, I'll create a docker container using the official httpd image, and I'll name this container, webserver:

```
docker run --detach --name webserver httpd
```

Ok now I have a running container. Also here's the equivalent kubectl run command for creating pod with a httpd container running inside it:

```
kubectl run webserver --image=httpd --restart=Never 
```

Normally I would create a pod using the kubectl apply command along with a yaml file, like the way I did in the hello world demo. But I've done it like this just to show what kubectl run is capable of. 

Ok we now have a running pod. Notice how similar these 2 commands are again. 



Now when using docker, you need to use the docker exec command to get inside a running container. 

```
$ docker exec -it webserver bash
pwd
```

This runs the bash command inside the webserver cotainer and it attaches my terminal to the resulting bash session as an interative terminal.


Now when it comes to Kubernetes, I can take a similar approach, using the kubectl exec command. 

```
$ kubectl exec -it webserver -- bash
```

Once again notice how similar these 2 commands are.

This practice of going inside a pod is often referred to execcing into a pod. 

You can also exec into a container just to run a single command and then exit out again:

```
$ docker exec -it webserver echo hello
$ kubectl exec -it webserver -- echo hello
```

Ok that's it for this video. See in you in the next video. 
