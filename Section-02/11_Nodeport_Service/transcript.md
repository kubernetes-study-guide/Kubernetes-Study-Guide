Hello everyone and welcome back. 

So far we have touched on nodeport services a few times already, but we haven't properly explained what they are and how they work. Generally speaking, Services are Kubernetes objects that helps with routing traffic to the correct pods. Services essentially sits in front of your pods and acts as a gateway to them. So when you're creating service objects your actually configuring your cluster's networking. 

We've already demoed the nodeport service a few times already, but in this video we're going to take a closer look at this service. We're going to use the same set of yaml files that we used in our previous demos . So let's start by creating our apache and centos pods:

```
code -f ...
kubectl apply -f 
kubectl get pods -o wide
```

Ok that's done, our pods are now running. Let's now  open up the nodeport service yaml file:

```
code config/svc.yaml
```

Ok let's create this service as well:

```
kubectl apply -f 
kubectl get services -o wide
```

Now Let's take a moment  to see what this service yaml   file is saying:

- Here we're saying, We want to create a service object 
- We've given our service a name, as we earlier, this name is also used to set up a dns record for this service      
- This service is going to be of the nodeport service type. There are other types of service   , such as clusterIP service, and we'll cover more about them later in the course. 
- Next we have set three port numbers:
-  port 3050 is the port that this service will listen on for internal traffic, for example other pods in the cluster.
````

Let's confirm that's the case curling this port from our centOS pod.

```
kubectl exec curl ...
```

ok so far so good. next we have the target port. 

- This    is the port number our servive will use to forward traffic to the destination pods. In this case it is set to port 80 since that's the port our apache container will be listening on. 
- Next we have The nodePort, this is the port number that all the worker nodes in the cluster will listen on for requests coming from outside the kubecluster. You are only allowed to use a port number from a specific range (30000-32767), also you can't have more than one nodeport service using the same nodeport number . The nodeport  setting is actually optional, so if leave it out, then kubernetes will  automatically look an available nodeport number and then assign it for you.  

Now how do we try out this nodeport number. 

My minikube provisioned kubecluster is running in a self contained VM on my macbook, that means my   kubecluster views my macbook as the outside world. So we can demo this nodeport setting by trying to access it from my  macbook.


```
curl http://${minikube ip}:30050
```



Notice here that this time we didn't curl our nodeport service's dns name, instead we had to use the minikube vm's ip address. That's because my macbook doesn't have access to Kubernetes DNS, since Kubernetes DNS is only available internally in the cluster    , but we can mimic it by adding an entry to our macbook's etc-hosts file:


```
vim /etc/hosts
cat /etc/host
minikube-ip-address nodeport-name  
curl http://nodeport-name:30050

```

Editing my macbooks hosts file like is something I sometimes do for troubleshooting problems.  


- last but not least, we arrive at The selector. This is a really important. That's Because this setting helps the service identify which pod it's allowed to forward traffic to. Here the selector says, only forward traffic to pods that have the label with the name of "app", along with the value of apache_webserver. Our apache pod has this matching label, so at this moment in time, our service should have our apache pod listed as an allowed destination, in otherwords it should be an active endpoint

We can confirm that's the case by viewing The service's active endpoints:

```

$ kubectl get endpoints -o wide
NAME                 ENDPOINTS           AGE
kubernetes           192.168.64.3:8443   12d
svc-nodeport-httpd   172.17.0.3:80       2m36s
$ kubectl get pods -o wide
```

endpoints lists out the available destinations for a service        . Here we can see that, as expected our apache pod's   ip address is on the allowed list of endpoints. 


 
So if you assumed that labels are just for storing some random bits of information, then you would be wrong. Labels plays a critical    role in configuring services.




So far I've demoed the nodeport service locally on my macbook, but how are nodeport services used in the real world .  

Well, one possible real world scenario, is that your cluster is    of EC2 instances running on the aws cloud platform. You have also registered your own domain name using a service like Godaddy, and you have configured your domain name using AWS route53  so that they point to your aws loadbalancer. This  loadbalancer then forwards traffic to your cluster's worker nodes. 

{lots of powerpoint slide here to show above diagram}





There's one final thing I wanted to bring up before I end this video, and that is that there are a lot of people who are put off from using   nodeport services. that's because of it's reliance of using non-standard ports. This means that you're forced to explicitly specify the nodeport number in your urls, like we did when we ran the curl command from our macbook. It doesn't look neat and it's a bit primitive. Also the more nodeport services you have, the more ports you're worker nodes ends up listening on, which isn't great from a security standpoint. 


That's why a lot of people use a combination of ClusterIP services and Ingress objects, as a powerful alternative to using nodeport services. We'll cover that later in the course. 

Ok that's it for this video. See you in the next one. 
  