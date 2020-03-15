# Hello World - Services

Structure: 
  vscode > slide > vscode



I'm now going to create a service object to make my pod externally accessible. 

In Kubernetes, Services are used for configuring networking, such as exposing a pod to outside traffic. 

There are different types of services, in our case we are going to create a NodePort service.

So let me open up the yaml files I'll be using in this demo:


```bash
# use vscode here. 
ls -l
code svc-nodeport.yml pod-httpd.yml 
```

Here we have 2 files. The one on the right is a copy of the pod yaml file from the last demo. The one on the left is for the new nodeport service we're going to create. 


Here we're saying:

- We want to create a service object 
- This service's name is going to be svc-nodeport-httpd
- this service is going to be of the type nodeport nodeport
- Next we have three port numbers.
-  port 3050 is the port number that the service itself will be listening on. 
- the target port is the container's port number that the service will forward traffic to.
- ... and The nodePort is the port number that all the kubeworker nodes will start listening on at the node OS level. See the course notes to learn more about these, and different use cases. 
- Finally we have The selector. This is a really important setting. that's becuase this is the setting that links this service to our hello-world pod. basically it says only send traffic to pods that have the label, app equal to apache_webserver




Now let's create these objects:

```bash
$ kubectl apply -f .
pod/pod-httpd created
service/svc-nodeport-httpd created
```

Notice this time I just specified a folder rather than a specific yaml file. This has the effect of applying all the yaml files in that folder. This ends up creating our apache pod from the last demo, and the service object. Now let's view our service:

```bash
$ kubectl get services -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          8d     <none>
svc-nodeport-httpd   NodePort    10.107.181.71   <none>        3050:31000/TCP   117s   app=apache_webserver
```

This 'kubernetes' service comes included as part of Kubernetes itself and it is used for internal purposes only.

Here we can see that our service now exists. It has the service name that we set in the yaml file. And it is a nodeport service. It has the correct port and nodeport that we asked for, and this service will forward all traffic to pods with the label we asked for. 

Let's bring up our pod along with it's labels:

```
kubectl get pods -o wide --show-labels
```

So based on our labels and selectors setup, our service should forward traffic to our pod. A quick way to check that is by listing the endpoints. 

```
kubectl get endpoints -o yaml
```

This says that any traffic going to xxx will be forwarded. 





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



let's now finish off by deleting our objects using the kubectl delete commands: 

```bash
$ kubectl delete -f .
pod "pod-httpd" deleted
service "svc-nodeport-apache-webserver" deleted
```





That's it for this video. See you in the next one. 
