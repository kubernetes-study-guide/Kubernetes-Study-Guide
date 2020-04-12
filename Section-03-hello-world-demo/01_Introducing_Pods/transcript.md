Getting a containerised application to run on kubernetes involves creating a set of kubernetes objects. There are different types of kubernetes objects, and you can list out the different types using the api-resources command.

```
minikube status
kubectl api-resources
```

As you can see there's a lot here. However there's one object type that's front and center of them all, and that the pod type. That's because pods are where you run your containers. Most of the other objects types plays more of a supporting role to your pods.

For example -

service objects can be used for routing network traffic to your pods.
secrets objects can be used for injecting sensitive data into your pods, such as username and passwords
configmaps objects can be used for supplying config files to the apps running inside your containers.
Persistant Volumes can be used to provide disk storage space for your pods to store data in.



We'll cover all this and more later on, and for now what I want you to take away from this is that pods are the fundamental building block in kubernetes.

We're now nearly ready to do our first kubernetes hello world demo, but before that I need to give an overview of the example app we'll be using for this demo. That's coming up next.