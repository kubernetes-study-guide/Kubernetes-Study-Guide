# hello-world pod

We're now ready to do our first hello world example by creating a single pod, since pods are the fundamental building block in Kubernetes.  

This pod will have a single container running inside it and this container will be our apache container. 

The first step is to create a yaml file that defines the pod's specifications. So here's a yaml file that I wrote earlier:

```bash
ls -l
cat pod-httpd.yaml
```

These yaml files follows a certain structure and a big part of Kubernetes is to do with writing these files. We'll cover how to write these files later.

But for now this yaml file is basically saying that:

- We want to create a pod
- The pod's name should be "webserver"
- we want to assign a key/value label, in this case I'm specifying a label which I'm going call 'app' and set that to the value of 'web'. I can add more labels here if I want to, but i'll stick with just the one label for this demo.
- This pod should only have one container
- the containers name should be 'cntr-apache'
- the container should be listening on port 80
- and finally, the container should be built using the official httpd image from the docker hub website. and also use the image that's tagged as 'latest'.



Let's now create this pod by feeding this yaml file into the apply command:

```bash
kubectl apply -f pod-httpd.yml
```

It doesn't matter what the filename is, as long as it's meaningful to you, and it ends with a dot yaml extension.

Ok it looks like our pod has now been created. Lets see if we can view it using the 'get pods' command:

```bash
kgp
```

This command lists all the pod in our kube cluster. In our case it shows  the pod that we've just created. The pod's name is the name that we specified in our yaml file. Our pod houses a total of one container, and here it says that 1 out of 1 container is in a ready state. The Pod is currently in running status and kubernetes hasn't had a need to restart it.

I'll set the output flag to 'wide' to print out some more info:

```bash
kgp -o wide
```

This shows a bit more info, in particular which worker node the pod is running on, as well as the pod's IP address. 

Now we know what our pod's ip address is, let's now try curl it see if we get a successful response from your apache container:

```bash
curl http://pods-ip-address
curl: (7) Failed to connect to 172.17.0.8 port 80: Operation timed out
```

Ok it timed out, let me try running the curl command again but this time from inside the kube cluster itself:

```bash
$ minikube ssh
$ curl http://172.17.0.8
```

This time it worked. That's because the pod's ip address is part of the kube cluster's internal network. So this curl command will only work if you run it from anywhere inside the kube cluster, such as from another pod, or from any kube worker nodes. 

Back outside the cluster you can also curl your pod by setting up port forwarding first:

```
exit
xxxx
```

Her'e we're saying that any traffic our workstations receives on port 80 should be forwarded to our apache pod at port 80

This is a bit like creating a tunnel from our workstation to our kube cluster. 


Let's hit return. 
This causes the terminal to hang, so we have to run curl from another terminal. 


```
curl 
```

Bingo that worked. 

Portfording is actually a technique used by software engineers for development purposes only. And it isn't the proper way to make your pod externally accessible. 


The proper way, is by using service objects, we'll cover that next. 

















