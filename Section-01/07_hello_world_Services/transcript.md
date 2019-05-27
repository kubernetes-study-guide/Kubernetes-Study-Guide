# Hello World - Services

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

We're now going to build on our previous hello-world example by making our pod accessible directly from our workstation. That's done by creating 'service' objects.

A service object is used to setup networking in our kube cluster. For example, if a pod exposes a web based gui, then a service object needs to be set up to make that pod's gui externally accessible.

There are different types of services, in our case we are going to create a NodePort service. NodePort services are quite crude and isn't recommended for production, but we're using it here because it's relatively easy to understand.

So here's the yaml file I'll be using to create the nodeport service:

```bash
tree configs/
code configs/pod-httpd.yml 
```

Here we're saying:

- We want to create a service
- and give that service a name
- And set the service type to nodeport
- Next we have three port numbers. the target port is the container's port number that the service will forward traffic to. 
- The nodePort is the port number that our kubecluster as a whole will start listening on at the node level, and these nodeport traffic will then get forwarded to the target port.
- Similarly the port number is for receiving traffic from other pods in the kubecluster and forwarding them to the target port.
- The selector section is another important setting. that's becuase it is the part that links this service to our pod. basically it says only send traffic to pods that have the label, app equal to apache_webserver




Now let's create this service object:

```bash
$ kubectl apply -f configs
pod/pod-httpd created
service/svc-nodeport-httpd created
```

Notice this time I just specified the parent folder. This has the effect of applying all the yaml files in that folder, which can be a big time saver. This ends up creating our apache pod from the last demo, and the service object. Now let's view our service:




$ kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          8d     <none>
svc-nodeport-httpd   NodePort    10.107.181.71   <none>        3050:31000/TCP   117s   app=apache_webserver
```

This 'kubernetes' service comes included as part of Kubernetes itself and is used for internal purposes only.

Now let's do a curl test:

```bash
$ curl http://$(minikube ip):31000
<html><body><h1>It works!</h1></body></html>
```

Here's another way to run the curl command:

```bash
$ minikube service svc-nodeport-httpd --url
http://192.168.99.107:31000

$ curl $(minikube service svc-nodeport-httpd --url)
<html><body><h1>It works!</h1></body></html>
```

Another cool thing, is that if you leave out the url flag then minikube will open up a web browser for you instead:

```bash
$ minikube service svc-nodeport-httpd
🎉  Opening kubernetes service default/svc-nodeport-httpd in default browser...
```

Now let's finish off by deleting our objects. We can delete everthing by just referencing the config folder that has all our yaml files: 

```bash
$ kubectl delete -f ./configs
pod "pod-httpd" deleted
service "svc-nodeport-apache-webserver" deleted
```

That's it for this video. See you in the next one. 
