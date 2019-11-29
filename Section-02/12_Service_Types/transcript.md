
There's one final thing I wanted to cover before I end this video, and that is that in Kubernetes there are several types of services, and nodePort type is just one of them. 
```
$ kubectl explain service.spec.type
```


If you only want internal traffic reaching your pods then you shouldn't use nodeport services, then you should use clusterip services. That's 

