Hello everyone, and welcome. 


In the last we saw how to use kubectl to create a new pod for the sole purpose of creating a bash terminal inside it. We also demoed a similar approach using docker. 

Now what if you want to create a bash session inside an existing container?

That's possible using docker exec, if you're working with containers directly, or with kubectl exec if the container is running inside a pod. Like last time, I'll demo both approaches side-dy-side so that you can see the similarities. 

Also like before, I'll start monitoring my list of containers and pods. 

```

```

As you can see we don't have any containers or pods running. 

For this demo, I'll create a docker container using the official httpd image, and I'll name this container, webserver:

```
docker run --detach --name webserver httpd
```

Ok now I have a running container. Also here's the equivalent kubectl run command for creating pod with a httpd container running inside it:

```
kubectl run webserver --image=httpd --restart=Never 
```

Normally I would create a pod using the kubectl apply command along witha yaml file. But I've done it using the kubeclt run command just to show that it's possible to do it this way. 

Ok we now have a running pod. Notice how similar these 2 commands are again. 



Now when using docker, you need to use the docker exec command to get inside an existing container. 

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

This practice of going inside a pod is often referred to execcing into a pod. It's a really handy technique to help troubleshoot problems. 
