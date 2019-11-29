Hello everyone and welcome back. 

So far we have touched demoed services a few times but we haven't properly explained what they are and how they work. Services are Kubernetes objects that helps with routing traffic to the correct pods. Services essentially sits in front of your pods and acts as a gateway to them. So when you're creating service objects your actually configuring your cluster's networking. 


There are several types of service objects. 

- nodeport
- clusterip 
- loadbalancer
- ingress

and in this video we're going to take a look at the nodeport service in more detail. We'll cover the others as we go through the course. 

We've already demoed the nodeport service when we talked about Kubernetes DNS. And we're going to use the same set of yaml files for this demo as well. So let's start by creating our apache and centos pods:

```
code -f ...
kubectl apply -f 
```

Ok that's done now, I'll just close the cento yaml file for now. Now le'ts open up the service file:

```
code config/svc.yaml
```

Ok let's create this service as well:

```
kubectl apply -f 
kubectl get services -o wide
```

Now Let's take a moment  to see what this file is saying:

- Here we're saying, We want to create a service object 
- This service is going to be called svc-nodeport-httpd
- This service is going to be a nodeport service.
- Next we have set three port numbers:
-  port 3050 is the port that this service will listen on for handling requests, for example other pods in the cluster.
````

Let's confirm that works. Now let's test this out.

```
kubectl exec curl ...
```

ok that worked. next we have the target port. 

- the target port is the port number that the service will use to forward traffic to the destination pods. In this case it is set to port 80 since that's the port our apache container will be listening on. If these port numbers don't match then we'll get an error message. 
- and we have The nodePort, this is the port number that all the worker nodes in the cluster will listen on for requests coming from outside the kubecluster. 



From our minikube's point of view, it views our macbook as external to it. So we can demo this nodeport setitng by trying to access it from our macbook.


```
curl http://${minikube ip}:30050
```

We can't use the dns name of our service name because kubernetes DNS only provides this service internally in the cluster, but we cna mimic it by adding an entry to our etc-hosts file:


```
vim /etc/hosts
cat /etc/host
minikube-ip-address nodeport-name  
curl http://nodeport-name:30050

```

this is a handy technique for doing development work. 



- Finally we have The selector. This is a really important. Because this is the part that helps the service work out which pods to forward traffic to. Here the selector says, only forward traffic to pods that have the label with the name of "app", along with the value of apache_webserver.


 
So if you were thinking that labels are just for storing some random bits of information, then you would be wrong. Becuase labels are needed for this labels&selectors concept to function.




So far I've done locally on my macbook, so what is the real world equivalent to using the nodeport services.  

Well, one possible real world scenario, is that your cluster is made of EC2 VMs running on the aws cloud platform. You have also registered your own domain using a service like Godaddy, and you use AWS route53 to configure your DNS to point to an aws loadbalancer. This loadbalancer in turn forwards traffic to your worker nodes. 

{lots of powerpoint slide here to show above diagram}


