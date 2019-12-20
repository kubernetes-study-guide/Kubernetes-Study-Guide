Hello everyone and welcome back. 

In all our demos so far we used a single apache webserver pod. However in practice one pod might not be enough to handle all the traffic, That's why it's common practice to create a set of replica pods and then loadbalance the traffic across them. Having replica pods also means that there's no single point of failure, so, if one pod dies for whatever reason, then the traffic can failover to the remaining replica pods. (show diagram)

Ok so for this demo, I want to create replicas pods with this pod definition:

```
code configs/pod.yaml
```

This spec is similar to the pod definitions I used in previous demos. The only thing I've tweaked is the container's startup command. Here, I'm basically overwriting the index.html, to include the pod's hostname. And after that I start the main apache webserver process. It'll become clear why I did that a bit later now. but for now let's just say that this is the pod I want to make replicas for.  





So how do we create a set of replica pods? Well one way to do that is by creating a replicaset object. Here's the replicaset I'll create:

```


```

Here, we're saying, create an object of the type replicaset, this replicaset should ensure that there are exactly 3 running pods in the set.

This RS will have control over any pods with this label. In this exmaple. this label has a key with the name 'app', and it's corresponding value is set to httpd_webserver. 

Next we have the template section. This section essentially has a pod-spec definition nested under it. The only bits that are left out are the kind, apiversion, and the  pod-name. So basically we're telling the replicaset to use this pod definition as a template to spin up the replica pods. 

One thing to note is that the replica pod's needs to have a label that allows the replicaset to have control over it. If we don't have a matching label then you'll get an error message when trying to create this replicaset. 

Now let's go ahead and create the replicaset:

```bash
kubectl apply -f xxxx
kubectl get replicasets -o wide
```

Here we can see the current status of our replicaset. we can also describe the replicaset to get more details about what it has done so for.

```
kubectl describe pods 
```
Here it says that it's creatd 3 pods. so Let's take a look at them. 




```
kubectl get pods -o wide --show-labels
```

As you can see, we now have our 3 pod cluster. The pods are named after the replicaset followed by a random string. The random string is their because kubernetes objects aren't allowed to have the same name. 

Now let's take a look at how to load balance traffic to them. That's done by using service. Services, by design comes with builtin loadbalancing. To see what I mean, let's create the followng nodeporet service:

```
code configs/nodeport.yaml
```

There's nothing new here, this is just a typical nodeport service. The key thing to note here is that we've set the selector to match the label the replicaset will assign to the pods it creates. So that means our nodeport has. 
























make sure you don't have more than replicaset mathcing hte same pod, or you can end up getting wierd behaviours

also replicasets have versatile selector logic. 

replicasets

why do pods have a random string in their name? 


In Kubernetes, you can create Kubernetes objects whose job is to inturn create other kubernetes objects. These types of objects are referred to as controller objects. 

In this video we're going to demo replicasets. Replicasets is a controller object and it's job is to create a set of pods with identical specs.

Now you might be wondering why would you want to have group of identical pods in the first place, there's 2 main reasons. loadbalancing and failover. 

Here's an example of a replicaset. 

```
code configs/replicaset.yaml
```

Let's breakdown what's in this yaml. First we have the common settings, including.....

this section is essentially a pod spec nested inside the replicaset. 



Replicasets doesn't only creates the pods, it also ensures the desired state. For example. here it says that we should have exactly 3 pods runnning at any given time. So if we delete a pod, then the replicaset straightaway creates a new pod to bring the desired state back to 3 pods.. 


In practice you will rarely need to create replicasets, instead you'll create them indirectly by creating a higher level object called deployments. 


So you might be wondering, why bother creating identical pods in the first place. There's 2 main reasons

- failover - if a pod dies for whatever reason, then another pod can take over, while the replicaset is busy buildinng replacement pod for the pod that died.  
- loadbalancing - One pod might not be enough to handle all the traffic, so we create multiple pods so the load gets distributed amongst them. 
 

To take advantage of failover and loadbalancing we need to use replicaset and service objects together. Let's demo that now. I've already created a replicaset so let's now create 

