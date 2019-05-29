# Hello World - Services

Structure: 
  vscode > slide > vscode



Hello everyone and welcome back. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

Earlier we built our hello-world pod but we couldn't curl to it from outside the cluster. 

In this video we're going to fix that problem by creating a 'service' object.

In Kubernetes, Services are used for configuring networking, such as, make a pod externally accessible. 

There are different types of services, in our case we are going to create a NodePort service. NodePort services are quite crude and isn't recommended for production, we'll explain why that is in a later video, but we're using going to create a nodePort service in this demo because it's relatively easy to understand.

So let me open up the yaml files I'll be using in this demo:


```bash
tree configs/
code configs/svc-nodeport.yml configs/pod-httpd.yml 
```

Here we have 2 files. The one on the right is a copy of the pod yaml file that we saw earlier as part of the previous video. The one on the left is for the new nodeport service that we're now going to create. 


Here we're saying:

- We want to create a service object 
- This service's name is goign to be svc-nodeport-httpd
- this service is going to be of the type nodeport nodeport
- Next we have three port numbers.
We'll explain these port numbers in more detail   a bit later in this video. But for now, let's just say that:
-  port 3050 is the port number that the service itself will be listening on. 
- the target port is the container's port number that the service will forward traffic to.
- ... and The nodePort is the port number that all the kubeworker nodes will start listening on at the node OS level.
- Finally we haveThe selector. This is a really important setting. that's becuase this is the setting that links this service to our hello-world pod. basically it says only send traffic to pods that have the label, app equal to apache_webserver




Now let's create these objects:

```bash
$ kubectl apply -f configs
pod/pod-httpd created
service/svc-nodeport-httpd created
```

Notice this time I just specified a folder rather than a specific yaml file. This has the effect of applying all the yaml files in that folder. This ends up creating our apache pod from the last demo, and the service object. Now let's view our service:

```bash
$ kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          8d     <none>
svc-nodeport-httpd   NodePort    10.107.181.71   <none>        3050:31000/TCP   117s   app=apache_webserver
```

This 'kubernetes' service comes included as part of Kubernetes itself and it  is used for internal purposes only.

Here we can see that our service now exists. It has the service name that we set in the yaml file. And it is a nodeport service. It has the correct port and nodeport that we asked for, and this service will forward all traffic to pods with the label we asked for. 



By the way if you want to get info for both our pod and service in a single command then you do this by using the 'get all' command:

```bash
$ kubectl get all -o wide
```



Now let's do a curl test from our workstation, which is external to our kube  cluster:

```bash
$ curl http://$(minikube ip):31000
<html><body><h1>It works!</h1></body></html>
```

This time it has worked without needing to go inside the kube cluster. Notice I had explicitly specify the nodeport number. Using custom port numbers like this is one of the drawbacks to using the nodeport service. 

Here's another way to run the curl command which saves you the trouble of looking up the nodeport number:

```bash
$ minikube service svc-nodeport-httpd --url
http://192.168.99.107:31000

$ curl $(minikube service svc-nodeport-httpd --url)
<html><body><h1>It works!</h1></body></html>
```

Also if you leave out the url flag then minikube will open up a web browser for you instead:

```bash
$ minikube service svc-nodeport-httpd
🎉  Opening kubernetes service default/svc-nodeport-httpd in default browser...
```

Ok we've now seen the nodeport service in action, but what's actually going on behind the scenes? Why do we have to use 3 different port numbers?

Let me pull up the following diagram to help explain what's going on. This diagram shows a kubecluster that has a total of 3 worker nodes.

When you create pod, it get's created on a particular worker node. But services on the other hand are cluster wide objects, which means that a single service object      span across all worker nodes. 

When you create a nodeport service, kubernetes ends up configuring each worker node at teh OS level to start listening on the Nodeport Number. It also sets up networking so that any incoming nodeport traffic get's forwarded to the nodeport service. In order for that to work, the nodeport service itself also needs to be listening on a port number. When the nodeport service receives traffic it then forwards it to a pod with label that matches the selector. In this example we have 2 pods that has the matching label this, In this scenario the nodeport service will also act as a loadbalancer to these pods. 

Also the nodeport service needs to know which port number on the pod (i.e. targetport) to forward the traffic too. That's where the targetport comes in. 

Now we also have this curl pod. This is just a dummy pod I've added just for performing some simple tests. In this scenario, our curl-pod can curl to either of these httpd pods using their respective ip addresses and port 80. Or it can curl via the service's ip address, in which case it needs to do it via port 3050. We'll demo that in a later video. 

In our minikube setup, we have a similar setup. It's essentially the same thing but with a single node instead of three, but the core concepts are still the same. 


Now let's finish off by deleting our objects. We can delete everthing by just referencing the config folder that has all our yaml files: 

```bash
$ kubectl delete -f ./configs
pod "pod-httpd" deleted
service "svc-nodeport-apache-webserver" deleted
```





That's it for this video. See you in the next one. 
