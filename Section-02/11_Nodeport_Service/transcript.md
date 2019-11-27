So far we have touched on nodeport service. However there are several types of service object and nodeport is just one of them. 

- nodeport
- clusterip 
- loadbalancer
- ingress

the key feature that makes nodeport services standout from the other services is that it's used for allowing outside traffic into the cluster. 

In the case of a minikube provisioned cluster, We can access these nodeport services from our macbook. To demo this, let me create the pod to test with:

Here I've opened up a bash terminal inside this video's topic folder. I've created the same videos as in the kubernetes DNS video.  

```
kubectl apply -f centospod -f apache-pod -f nodeportservice. 
```

Let's take a minute to see what's going on here:

- Here we're saying, We want to create a service object 
- This service is going to be called svc-nodeport-httpd
- This service is going to be a nodeport service. There are other service types available, and we'll cover them later in the course.

- Next we have set three port numbers:
-  port 3050 is the port that this service will listen on for handling requests, for example other pods in the cluster.  
- the target port is the port number that the service will use to forward traffic to the destination pods. In this case it is set to port 80 since that's the port our apache container will be listening on. 
- ... and we have The nodePort, this is the port number that all the worker nodes in the cluster will on for requests coming from outside the kubecluster. 
- Finally we have The selector. This is a really important. It's the mechanism that links this service to our apache pod. basically it says only forward traffic to pods that have the label with the name of "app", along with the value of apache_webserver. 
 
So if you were thinking that labels are just for storing some information, then you would be wrong. Becuase labels are needed for this labels&selectors concept to function.


```
curl http://${minikube ip}:30050
```

We can even curl our nodeport's service name by updating our mac's etc hosts file with a new entry that resolves our nodeport name to our minikube's ip address.

```
cat /etc/host
nodeport-name   minikube-ip-address
curl http://nodeport-name:30050

```

this is a handy technique for doing development work. 


But that's all done locally on my macbook, so what is the real world equivalent to using the nodeport services.  

Well, one possible real world scenario, is that your cluster is made of EC2 VMs running on the aws cloud platform. You have also registered your own domain using a service like Godaddy, and you use AWS route53 to configure your DNS to point to an aws loadbalancer. This loadbalancer in turn forwards traffic to your worker nodes. 

{lots of powerpoint slide here to show above diagram}




