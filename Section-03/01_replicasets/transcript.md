replicasets

replicasets are a kubernetes objects. They are controller objects. 

In Kubernetes, you can create Kubernetes objects whose job is to inturn create other kubernetes objects. These types of objects referred to as controller objects. 

In this video we're going to demo replicasets. Replicasets is a controller object and it's job is create a set of identical pods. 

Here's an example of a replicaset. 




Replicasets don't only creates the pods, it also ensures the desired state. For example. here it says that we should have exactly 3 pods runnning at any given time. So if we delete a pod, then the replicaset automatically creates a pod to bring back to the desired state. 




In practice you will rarely need to create replicasets, instead you'll create them indirectly by creating a higher level object called deployments. 


