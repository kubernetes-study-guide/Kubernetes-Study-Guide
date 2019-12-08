Hello everyone, and welcome back. 

So far we've only looked at one type of service, and that's the nodeport service. However there are a few other service types, and you can list them out using the explain command. 

```
$ kubectl explain service.spec.type
```

I'll give demos for these service types as we go through the course. But for now I just wanted to give you a high level overview of them. so by the end of this video, you'll have a better understanding what these service type      are, how they are different from each other, and under what circumstances one service type is  better suited over another.

Now we've already seen how the nodeport service works, As illustrated in this diagram. 

Here we saw that we needed to setup    a load balancer to allow outside traffic into our kube   cluster. 

In this example we have our kube cluster on AWS. So a typical way to create this AWS loadbalancer would by via the aws web console. However you can also get Kubernetes to create this loadbalancer for you. And that's what the loadbalancer service type is  for. That means that you can write out a loadbalancer service yaml file, and then just apply it, in the same way that you do with any  other yaml files. 

I personally find the loadbalancer service type a little     . That's because they effectively make Kubernetes manage objects that live outside of the cluster. All the yaml definitions we've encountered so far were to do with building objects inside the cluster, so loadbalancer service   falls out-of-step with that.

Next we have clusterIP services. 

Let's say you don't want to expose your pods to the outside world. In that scenario you won't need any   loadbalancers. And for that matter, also don't want your service listening for outside traffic via the nodeport number. That's where clusterIP services comes to the rescue. 

ClusterIP services are similar to nodeport services. The key difference is that clusterIP services don't listen for external traffic. In otherwords, clusterIP services restricts access to pods to internal cluster traffic only. That's why, I often think of clusterIP services as a cut down version of nodeport services.

ClusterIP is actually the default service type, meaning that if you don't explicitly specify the service type in your yaml file then kubernetes will assume it to be a clusterIP service.

Theres another object type that's often used in conjunction with clusterIP services. And that's Ingress objects. Ingress objects provides another way to allow external traffic into your cluster. You can    configure your ingress objects to forward the outside traffic to one or more clusterIP services.  

This clusterIP and ingress combo is a really powerful and popular alternative to using nodeport services. One reason for that is that unlike nodeport services, this combo let's you're kube cluster accept traffic on standard port numbers. We'll go into more detail about this,  later in the course. 

And finally we have the externalname Service.


Now there can be situations where you want your pods to reach out and access a resource outside the cluster, for example, a mysql database. In this example our mysql database has quite a complicated connection string.

We can use this connection string to interact with the mysql db. However a better approach would be to use a nicer, human readable connection string. Enter ExternalNames services. 

Put simply, externalname services are used for adding custom c-name entries to Kubernetes DNS. So in this scenario you can write a yaml file for your ExternalName service associates youre new dns name with the mysql db's connection string. Then apply that yaml file so that your new dns entry get's added to Kubernetes DNS. After that, Your pods can then use the new connection string instead of the complicated one. 

Ok, That's it for this video, see you in the next one. 






