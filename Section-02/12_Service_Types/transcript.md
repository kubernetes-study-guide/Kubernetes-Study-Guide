So far we've only looked at nodeport services. HOwever there are a few other service types



There's one final thing I wanted to cover before I end this video, and that is that in Kubernetes there are several types of services, and nodePort type is just one of them. 
```
$ kubectl explain service.spec.type
```


If you only want internal traffic reaching your pods then you shouldn't use nodeport services, then you should use clusterip services. That's 

By the way, this loadbalancer is something that sits outside the cluster, but you can actually create this loadbalancer by create a loadbalancer service. This is a bit of a weird feature of Kubernetes becuase it's one of those rare occasion where it's making changes outside of the cluster