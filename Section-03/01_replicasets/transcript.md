replicasets


In Kubernetes, you can create Kubernetes objects whose job is to inturn create other kubernetes objects. These types of objects are referred to as controller objects. 

In this video we're going to demo replicasets. Replicasets is a controller object and it's job is to create a set of pods with identical specs. 

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

