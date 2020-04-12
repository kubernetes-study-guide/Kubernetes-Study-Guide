Getting a containerised app running on kubernetes involves creating a set of kubernetes objects. There are different types of objects that you can create, and you can list them out using the api-resources command.

```
minikube status
kubectl api-resources
```

As you can see there's a lot here. However there's one object type that's front and center of them all, and that the pod type. That's because pods are where you run your containers. Most of the other objects types plays more of a supporting role to your pods.

For example -

services are used for routing network traffic to your pods.
secrets are used for injecting sensitive data into your pods, such as username and passwords
If you want to provide config files to the apps running inside the containers, then you can do that by injecting configmaps to your pods.
Persistant Volumes are useful for allocating disk space for your pods to store data in.



We'll cover all this and more later on, but for now, what I want you to take away from this video. is that pods are the fundamental building block in kubernetes.

