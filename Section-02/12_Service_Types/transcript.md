Hello everyone, and welcome back. 

So far we've only looked at one type of service, and that's the nodeport service. However there are a few other service types, which you can list out using the explain command. 

```
$ kubectl explain service.spec.type
```

I'll give a demo for these service types as we go through the course. But for now I just wanted to give you a high level overview of them so that you start to get familiarised with them and how they all fit in.

Now we've already seen how the nodeport service works, As illustrated in this diagram. 

And in this diagram we saw that we we needed to create a load balancer to allow outside traffic into the cluster. 

In this example we have our kube cluster on AWS. So a typical way to create this AWS loadbalancer would by via the aws web console. However you can also get Kubernetes to create this loadbalancer for you. And that's what the loadbalancer service type is used for. That means that you can write out a loadbalancer service yaml file, and then just apply it, like you do with everything.

I personally find loadbalancer service type a bit wierd. That's because it means that your're effectively getting Kubernetes to manage a resource that lives outside the cluster. All the yaml definitions we've encountered so far were to do with building things inside the cluster, so loadbalancer services is out of step with that.

Now what if don't want to expose your pods to the outside world. In that scenario you would not want an exteranl loadbalancer. Also using a nodeport service wouldn't be suitable either, because they are designed to allow outside traffic via the nodeport port number. That's where the clusterIP service type into the picture. 

The clusterIP services are similar to nodeport services. The key difference is that clusterIP services don't listen for external traffic. In otherwords, clusterIP services are used to makes pods accessible to internal cluster traffic only. This is actually the most commonly used service type, which is why it's the default service type. So if you don't specify the service type in your yaml definition then kubernetes will assume it to be clusterIP. 

Now there can be situations where you want your pods to access something outside the cluster, for example, an ftp server. Now let's say that this ftp server doesn't have an dns address, it just has a puplic ip address.  

In that scenario, you can let your pods use that ip addresss to reach ftp server. However as explained earlier


ingress objects. 











I'm going go back to our earlier nodeport diagram that I showed you earlier. 



Now in this diagram we had to create the aws loadbalancer to distribute traffic to our worker nodes. 


ClusterIP is actually the default service. So if you leave out this line then kubernetes will default to creating this as a clusterIP service. 




There's one final thing I wanted to cover before I end this video, and that is that in Kubernetes there are several types of services, and nodePort type is just one of them. 



If you only want internal traffic reaching your pods then you shouldn't use nodeport services, then you should use clusterip services. That's 

By the way, this loadbalancer is something that sits outside the cluster, but you can actually create this loadbalancer by create a loadbalancer service. This is a bit of a weird feature of Kubernetes becuase it's one of those rare occasion where it's making changes outside of the cluster