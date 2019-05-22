# hello-world pod

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

In this video we're going to create our first hello-world app. This app is going to be a pod with an apache webserver container running inside it. 

To create this pod we first need to create a yaml file the defines the pod's specifiction. So here's the yaml file I'll be using:

```bash
tree configs/
code configs/pod-httpd.yml 
```

These yaml files follows a certain structure which I'll go into more detail later. But for now this yaml file is basically saying that:

- We want to create a pod
- The pod's name should be pod-httpd
- we want to assign a label to the pod of 'app equal to apache_webserver'
- This pod should only have one container
- the containers name should be cntr-httpd
- the container should be built using the latest official httpd image from the docker hub website. 
- and finally, the container should be listening on port 80

Let's now create this pod by feeding this yaml file into the apply command:

```bash
kubectl apply -f configs/pod-httpd.yml
```

By the way, it doesn't matter what you set the yaml file's filename as, as long as it's meaningful to you, and it ends with a dot yml extension. 

Ok it looks like our pod has now been created. Lets now see if we can view it using the 'get pods' command:

```bash
kubectl get pods 
```

If you have a lot of pods, and just want get the info for one pod, then you can specify the pod's name in your command:

I'll set the output flag to 'wide' to print out more info:

```bash
kubectl get pods -o wide 
```

I can also display the labels using the --show-labels flag:

```bash
kubectl get pods -o wide --show-labels
```

However, if we want to get all the info about our pod, then we need to set the output flag to 'yaml':


```bash
kubectl get pods pod-httpd -o yaml
```

Also notice that I specified the pod's name in the command. That's to tell kubectl to only retrieve info for that one pod. 

Another command that gives a lot of info is the describe command:

```bash
kubectl describe pod pod-httpd
```

Let's now clear the screen and return back to the get command:

```bash
clear
kubectl get pods -o wide 
```

So far, we've created a single-container pod. This container is supposed to have the apache webserver running inside it. But how do we check if that web service is definitely working? 

One thing you might think of trying, is to curl the pod's ip address:

```bash
curl http://pod-ip
```

However that's not going to work, That's because the pod's ip address is part of the kube cluster's internal network. Kubernetes only comes with some basic networking features out-of-the-box. Those networking features lets you curl the pod's ip:

- from inside the container itself
- or from another container in the kubecluster
- or from one of the nodes that make up the kubecluster. 

So let's try performing a curl test from inside the apache container itself. To do that we'll open up a bash session inside the apache container. That's done by using the exec command:

```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
root@pod-httpd:/usr/local/apache2#
```

This command is similar to the docker exec command. The -it flag says that we want to create an interactive terminal. The -c flag says which container inside the pod we want to connect to. And everything after the double dash, says what command we want to run, which in our case is to start a bash session. 

Once we're inside the container, we then need to install curl, and in our case, we do that by running apt-get:

```bash
apt-get update
apt-get install curl
```

Now let's run the curl test:

```bash
root@pod-httpd:/usr/local/apache2# curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

Here we can see that it's working, so let's exit out of the container. 

By the way, you can also run a command inside a container without going into it first, all you have to do is remove the -it flag, for example here's an exec command that prints out the container's hostname:

```bash
$ kubectl exec pod-httpd -c cntr-httpd -- hostname
pod-httpd
```

Now let's try to curl our web service from inside the minikube vm. First let's remind ourselves what the pods ip address is:

```bash
kubectl get pods -o wide
```

Now let's ssh into the minikube vm and curl against this ip address. 

```bash
$ minikube ssh
curl http://pod-ip
exit
```

So that looks like it's working as well. let's exit out now.

We've now completed all our testing so we'll finish up by deleting the pod.

```bash
$ kubectl delete pod pod-httpd
pod "pod-httpd" deleted
```

Ok that's it for this video see you in the next one.