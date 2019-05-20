
In this video we're going to create our first cube cluster using minikube
First let's check minikube's status.

Here we can see that we don't have a kube cluster running at the moment. 



So let's create a new kube cluster by running minikube start. This can take a few minutes to complete so we'll fast forward this to save you from waiting. 


Ok thats done now minikube has provided a summary of all the tasks it performed as part of the cluster build process. There's quite a lot of interesting info here. Here it shows that minikube downloaded an image file and used that image file to create a VM using Virtualbox. This vm has this amount of cpu, ram and disk space allocated to it. You can confirm that's the case by taking a look in virtualbox, notice that a VM with the name minikube now exists with the spec we saw earlier.  


Moving on, It shows what IP address the VM has been assigned with. It also shows that the kubecluster is using docker as it's container run time engine.  




 
now let's check mini kube status again. Here it says that our local kubectl client has been configured to point this cluster.


Let's confirm that's the case by now actually trying to use kubectl. First I'll try getting the cluster-info. So far so good. This ip address matches up with the minikube vm's ip address (minikube ip)


Now let's perform a component status health check. 

Now let's get a list of all the nodes in our kube cluster.

```
kubectl get nodes
```

We can get more info by setting the output flag as wide. 

```
kubectl get nodes -o wide
```

here we can only see one node listed, which is expected since minikube only builds single node kube clusters. 

Ok, let's clear the screen now. 

Ok let's go back to minikube because there are a few other things I wanted to show you. 
If you run minikube-help then you'll see all the available commands. 

```bash
minikube
```

Here we can see that if you want to ssh into the minikube vm, you just run:

```bash
minikube ssh
```

After you're logged you can then switch to root user. ok I'll exit out now. 

In kubernetes there is an app you can install called Kubernetes Dashboard, This provides 
a powerful graphical web interface. And the really cool thing with minikube is that this dashboards comes preinstalled.
You can access the web ui by running the dashboard command:

```bash
minikube dashboard
``` 

Finally once you have finished working, you can run:

```bash
minikube stop
```

This will shutdown your minikube vm. However if you want delete it altogether then you use the delete command:

```bash
minikube delete
```

That's it for this video see you in the next one. 