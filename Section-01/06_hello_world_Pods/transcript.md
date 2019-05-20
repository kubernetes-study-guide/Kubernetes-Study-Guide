# hello-world pod

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

In this video we're going to create our first hello-world app running on Kubernetes. This app is going to be an apache webserver container that's housed inside a pod. 

To create this pod we first need to create a yaml file the defines the pod's specifiction. So here's the yaml file I'll be using:

```bash
code configs/pod-httpd.yml 
```

These yaml files follows a certain structure which I'll go into more detail later. But for now this yaml file is basically saying that:

- We want to create a pod
- The pod's name should be pod-httpd
- assign a label to the pod of 'app equal to apache_webserver'
- This pod should only have one container
- the containers name should be cntr-httpd
- the container should be built using the latest httpd image from the docker hub website. 
- finally the container should be listening on port 80

Let's now create this pod by feeding this yaml file into the apply command:

```bash
kubectl apply -f configs/pod-httpd.yml
```

By the way, it doesn't matter what you set the yaml file's filename is, as long as it's meaningful to you, and it ends with a .yml extension. 

Ok it looks like our pod has been created. Lets now see if we can view it using the get pods command:

```bash
kubectl get pods 
```

I'll set the output flag to 'wide' to print out more info:

```bash
kubectl get pods -o wide 
```

I'll add on the show-labels flag as well to view the labels too:

```bash
kubectl get pods -o wide --show-labels
```

However to get all the available info for our pod, we need to set the output flag to 'yaml':


```bash
kubectl get pods pod-httpd -o yaml
```

Whenever you come across a command where you specify an object type following by an objects name. 


There's also the describe command that also gives detailed info:














