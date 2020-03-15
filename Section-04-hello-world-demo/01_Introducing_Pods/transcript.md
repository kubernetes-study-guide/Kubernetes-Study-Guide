Setting up an containerised application to run on kubernetes involves creating a set of kubernetes objects. There are a lots of different types of kuberentes object, such as services, replicicasets, replicates,....and so on. However there's one object that plays front and center out of all these types and that's, pods. That's because pods are where you run your containers. [A single pod can have one or more containers running inside it.] 


All the other objects types plays more of a supporting role to your pods. 

For example -

service objects can be used for routing network traffic to your pods. 
secrets objects can be used for injecting secrets into your pod, such as username and passwords 
configmaps objects can be used for providing initial configurations for the applications running inside the container in your pod.
Persistant Volumes can be used to provide disk storage space for your pods to store data in. 



We'll cover all this and more later on, and for now what I want you to take away from this is taht pods are the fundamental building block in kubernetes.