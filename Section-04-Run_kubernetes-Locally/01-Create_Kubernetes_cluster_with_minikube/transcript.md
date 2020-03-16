
In this video we're going to create our first Kube cluster using minikube
First let's check minikube's status.

Here we can see that we don't have a kube cluster running at the moment. 



So let's create a new kube cluster by running minikube start. This can take a few minutes to complete so we'll fast forward this to save you from waiting. 


Ok thats now done. minikube has given a summary of what it did to build the kube cluster. There's a lot of interesting info here. Here it shows that minikube downloaded an image file and then used it to create a VM using Virtualbox. It shows how much cpu, ram, and disk space allocated to this VM. You can confirm that's the case by taking a look in virtualbox, As you can see a VM called minikube now exists with the same specs that we saw earlier.  


Moving on, It shows what IP address the VM has been assigned with. It also shows that the kubecluster is using docker as it's container runtime engine.  




 
now let's check minikube status again. 

```bash
minikube status
```

Here it says that our local kubectl client has been configured to point this new kube cluster.


Let's confirm that's the case by now actually trying to use kubectl. First I'll try getting the cluster-info. 

```bash
kubectl cluster-info
```



So far so good. This ip address matches up with the minikube vm's ip address:

```bash
minikube ip
```


Now let's perform a component status health check. 

```bash
$ kubectl get componentstatuses
```

Everything looks good here too. Now let's get a list of all the nodes in our kube cluster.

```
kubectl get nodes
```

We can get more info by setting the output flag to wide. 

```
kubectl get nodes -o wide
```

here we can only see one node listed, which is expected since minikube only builds single node kube clusters. 

Ok, let's clear the screen now. 

I think we can now safely say that our kubectl client is definitely pointing to our minkube built kubecluster. There are a few other things I wanted to show you about minikube. 


If you run minikube-help then you'll see all the available commands. 

```bash
minikube help
```

Here we can see that if you want to ssh into the minikube vm, you just run:

```bash
minikube ssh
```

After you're logged in, you can then switch to root user if you want. 

```bash

```

ok I'll exit out now. 

In kubernetes there is a really cool app you can install called Kubernetes Dashboard, This provides 
a powerful graphical web interface that let's you view and manage your kubecluster visually. This dashboard is quite straight forward to install. However the nice thing about minikube is that this dashboards comes preinstalled.
You can access the web ui by running the dashboard command:

```bash
minikube dashboard
```

I recommend exploring this dashboard as you go through the course. 

Finally once you have finished working with your kube cluster, you can run the stop command:

```bash
minikube stop
```

This will shutdown your minikube vm. However if you want to delete it altogether then you use the delete command:

```bash
minikube delete
```

That's it for this video see you in the next one. 