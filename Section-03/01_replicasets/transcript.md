Hello everyone and welcome back.

Wow! You made it to section 3 of the course! That's awesome! If you've been 


We're now going to level up and introduce you to a new type of objects called controller objects. Controller objects are kubernetes objects that in turn creates other kubernetes objects. A few examples of controller objects are replicasets, deployments, and statefulsets. We'll cover all of them in due course, but for now we'll take a look at replicasets.

A replicaset is used for creating multiple replica pods, hence the name replicaset. You need to create replica pods if you want to implement failover and loadbalancing. We'll cover more about that when we take a look at deployment objects later on in the course. 

But for now here's the replicaset we'll be creating in this demo:

```
code configs/replicaset-httpd.yaml

```


Here, we're saying, create an object of the type replicaset, this replicaset should ensure that there are exactly 3 running pods in the set.

This RS will have control over any pods with this label. In this exmaple. this label has a key with the name 'app', and it's corresponding value is set to httpd_webserver. 

Next we have the template section. This section essentially has a pod-spec definition nested under it. The only bits that are left out are the kind, apiversion, and the  pod-name. So basically we're telling the replicaset to use this pod definition as a template to spin up the replica pods. 

This spec is similar to the pod definitions I used in previous demos. The only thing I've tweaked is the container's startup command. Here, I'm basically updating the index.html file, to include the pod's hostname. And after that I start the main apache webserver process.


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



























Replicasets doesn't only creates the pods, it also ensures the desired state. For example. here it says that we should have exactly 3 pods runnning at any given time. So if we delete a pod, then the replicaset straightaway creates a new pod to bring the desired state back to 3 pods. 

Now in case you want to only have a single pod running, then you can create that pod using a pod yaml file. But even in that scenario it's still better to create it using a controller object such as a replicaset. That's because you can take advantage of self-healing. 





In practice you will rarely need to create replicasets, instead you'll create them indirectly by creating a higher level object called deployments. 








## References

https://kubernetes.io/docs/concepts/workloads/controllers/

https://www.mirantis.com/blog/kubernetes-replication-controller-replica-set-and-deployments-understanding-replication-options/