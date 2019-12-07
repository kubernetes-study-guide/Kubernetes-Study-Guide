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

Ok finally we have the externalname Service.


Now there can be situations where you want your pods to reach out and access a resource outside the cluster, for example, an ftp server. Now let's say that this ftp server has a puplic ip address but no dns name.  

Then in that scenario, you can let your pods use that ip addresss to reach the ftp server. However that's not good practice. If your ftp server's ip address changes in future, then your pods would need to be updated with the new ip address. A much better approach is to add a custom dns entry for your SFTP server into Kubernetes DNS. 

And that's where the externalname service comes into the picture. Put simply, externalnames services are used for adding custom dns entries to Kubernetes DNS. Your pods then can use the dns name instead of the ip address. So if the ip address does change, then you just need to update the single externalname service with the new ip.

Ok, That's it for this video, see you in the next one. 






