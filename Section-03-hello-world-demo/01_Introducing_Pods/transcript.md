Setting up an containerised application to run on kubernetes involves creating a set of kubernetes objects. There are different kinde of objects, such as services, replicasets, serviceaccounts,....and so on. However there's one object type that's front and center of them all, and that the pod type. That's because pods are where you run your containers. Most of the other objects types plays more of a supporting role to your pods. 

For example -

service objects can be used for routing network traffic to your pods. 
secrets objects can be used for injecting secrets into your pod, such as username and passwords 
configmaps objects can be used for providing initial configurations for the applications running inside the container in your pod.
Persistant Volumes can be used to provide disk storage space for your pods to store data in. 



We'll cover all this and more later on, and for now what I want you to take away from this is taht pods are the fundamental building block in kubernetes.

We're now nearly ready to do our first kubernetes hello world demo, but before that I need to give an overview of the example app we'll be using for this demo. That's coming up next. 