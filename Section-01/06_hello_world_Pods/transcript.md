# hello-world pod

Hello everyone and welcome back. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

In this video we're going to create our first hello-world pod. This pod is going to have an apache webserver container running inside it. 

To create this pod we first need to create a yaml file that defines the pod's specifications. So here's a yaml file that I wrote earlier:

```bash
tree configs/
code configs/pod-httpd.yml 
```

These yaml files follows a certain structure and a big part of Kubernetes is to do with writing these files. We'll cover how to write these files later. 

But for now this yaml file is basically saying that:

- We want to create a pod
- The pod's name should be pod-httpd
- we want to assign a key/value label to the pod where the key is set to 'app' and the value is set to 'apache_webserver'
- This pod should only have one container
- the containers name should be cntr-httpd
- the container should be listening on port 80
- and finally, the container should be built using the official httpd image from the docker hub website. and also use the image that's tagged as 'latest'. 

## TODO - Start
Theres one thing I should mention about this image setting, this is something that I looked up by going to the docker hub website, and then searched for apache. From there I found the image's name, along with available tags. If you leave out the tag, then it will just default to the 'latest' tag. 

I also omitted the full fqdn, which is why it defaulted to docker hub. 

## TODO - END. 


Let's now create this pod by feeding this yaml file into the apply command:

```bash
kubectl apply -f configs/pod-httpd.yml
```

By the way, it doesn't matter what the yaml file's filename is, as long as it's meaningful to you, and it ends with a dot yml extension. 

Ok it looks like our pod has now been created. Lets now see if we can view it using the 'get pods' command:

```bash
kubectl get pods 
```

This command lists all the pod in our kube cluster. In our case it shows  the pod that we've just created. The pod's name is the name that we specified in our yaml file. Our pod houses a total of one container, and here it says that 1 out of 1 container is in a ready state. The Pod is currently in running status and kubernetes hasn't had a need to restart it. 

I'll set the output flag to 'wide' to print out some more info:

```bash
kubectl get pods -o wide
```

This shows a bit more info, in particular which worker node the pod is running on, as well as what is the pod's IP address. That leads me onto the next thing we need to start looking at, we need to check if our pod is definitely working. One obvious way to test that is by trying to curl the pod's ip address. 

```bash
curl http://pod-ip
curl: (7) Failed to connect to 172.17.0.8 port 80: Operation timed out
```

However that appears to hang, which you might find a bit worrying. But before you get too concerned, let me try running the curl command again but this time from inside the kube cluster:

```bash
$ minikube ssh
$ curl http://172.17.0.8
```

This time it worked. That's because the pod's ip address is part of the kube cluster's internal network. So this curl command will only work if you run it from somewhere inside the kube cluster, such as from another pod, or from any kube worker nodes. 

The main way to make a pod externally accessible, is by creating a service object, we'll do in the next video.  

However there are a few other things I wanted to show you in this vidoe before we start creating service objects. First, you can run commands inside your pod using the exec command:

```bash
kubectl exec pod-httpd -c cntr-httpd -- ls -l
```

Here, the -c flag says which container inside the pod we want to connect to. And everything after the double dash, tells kubectl what command we want to run inside the container. If we omit the double dash then kubectl could treat the -l flag as a kubectl flag rather than the a flag for the ls command . 

Another important feature of the exec command is that it let's you start an interactive bash session inside your pod:


```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
root@pod-httpd:/usr/local/apache2#
```

This command is similar to the equivalent docker exec command. The -it flag says that we want to create an interactive terminal. Once your inside, you can run another curl test, but we first need to install curl:


```bash
apt-get -y update
apt-get install -y curl
```

Now let's run the curl test:

```bash
root@pod-httpd:/usr/local/apache2# curl http://localhost
<html><body><h1>It works!</h1></body></html>
root@pod-httpd:/usr/local/apache2# curl http://127.0.0.1
<html><body><h1>It works!</h1></body></html>
root@pod-httpd:/usr/local/apache2# hostname --ip-address
172.17.0.8
root@pod-httpd:/usr/local/apache2# curl http://172.17.0.8
<html><body><h1>It works!</h1></body></html>
```

Ok so everything looks good here, let me exit it out now. 

```bash
exit
```


Finally I wanted to show you some commands you can use to get more detailed info about your pods. The first one is the 'get pods' command again, but this time we set the output flag to 'yaml':


```bash
kubectl get pods pod-httpd -o yaml | less
```

Also notice that I specified the pod's name in the command. That's to tell kubectl to only retrieve info for that one pod.


This yaml output is essentially the full form version of the yaml file that we used to build this pod. It shows a lot of the defaults that were used since we didn't explicitly specify all these setting in our yaml file. 

Another command that gives a lot of info is the describe command:

```bash
kubectl describe pod pod-httpd | less
```

This has a lot of the same output as we saw with the get command. However it does have some other interesting info, such as an event log at the bottom, this can be useful for troubleshooting.  

Once you have finished using the pod, you can then delete it by using the 'delete' command:

```bash
kubectl delete pod pod-httpd
```

This can take a few seconds to complete. 


Ok that's it for this video see you in the next one.