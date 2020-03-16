# Hello World - Services

We're now going to improve this hello-world example by making our pod accessible directly from our macbook's web browser. That's done by creating a 'service' object.

**Service:** A service object is used to setup networking in our kube cluster. E.g. if a running pod exposes a web based gui, then a service object needs to be set up to make that pod's gui externally accessible.

So far we have only created the hello-pod:

```bash
$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-httpd   1/1     Running   0          29m   172.17.0.8   minikube   <none>           <none>
```


So in our hello-world example, we've created a new yaml file with the content:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport-httpd
spec:
  type: NodePort   # there are 4 types of Services. ClusterIP, NodePort, LoadBalancer, Ingress. https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types. NodePort should only be used for dev environments.
  ports:
    - port: 3050  # this is used by other pods to access assets that's avialable in our demo conainer
      targetPort: 80 # port number of the pod's primary container is listening on. So needs to mirror containerPort setting as defined in the object config file.
      nodePort: 31000  # this ranges between 30000-32767. Our worker node VM will be listening on this port. It's actually the kube-proxy component on worker nodes that will start listening on this port. This is the port number we need to enter into our web browser. That's one of the drawbacks in using nodePort service type, i.e. have to explicitly specify ugly port numbers in the url
  selector:
    app: apache_webserver  # this says it will forward traffic to object that has metadata.label entry with key/value pair of 'app: apache_webserver' that's how this object and the pod object links together.
```

There are different types of services, in our case we are creating a NodePort service. NodePort services are quite crude and isn't recommended for production, but we're using it here because it's the easiest service type to understand for a beginner. Now let's create the service object:

```bash
$ kubectl apply -f configs/svc-nodeport.yml
service/svc-nodeport-httpd created

$ kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          8d     <none>
svc-nodeport-httpd   NodePort    10.107.181.71   <none>        3050:31000/TCP   117s   app=apache_webserver
```

> The 'kubernetes' service comes included as part of the Kubernetes itself and is used for internal purposes only.

**Handy Tip**: So far we had to run the apply command twice so far, once for each yaml file. Luckily there's a way to apply all the configs in one command by simply specifying the directory that houses all your configs, e.g.:

```bash
$ kubectl apply -f configs
pod/pod-httpd created
service/svc-nodeport-httpd created
```

Similarly here's way to view all your objects:

```bash
$ kubectl get all -o wide
NAME            READY   STATUS              RESTARTS   AGE   IP       NODE       NOMINATED NODE   READINESS GATES
pod/pod-httpd   0/1     ContainerCreating   0          3s    <none>   minikube   <none>           <none>

NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          8d    <none>
service/svc-nodeport-httpd   NodePort    10.101.129.54   <none>        3050:31000/TCP   3s    app=apache_webserver
```

Next, you need to find the ip address of your worker node, which you can find by running:

```bash
$ minikube ip
192.168.99.100
```

Now you know the ip number and port number that you should be using, you can now test the endpoint right from your macbook, with curl:

```bash
$ curl http://192.168.99.100:31000
<html><body><h1>It works!</h1></body></html>
```

Another more shorthand way to run the curl command:

```bash
$ minikube service svc-nodeport-httpd --url
http://192.168.99.107:31000

$ curl $(minikube service svc-nodeport-httpd --url)
<html><body><h1>It works!</h1></body></html>
```

You can also test this endpoint by opening it up in your Macbook's web browser, just do:

```bash
$ minikube service svc-nodeport-httpd
🎉  Opening kubernetes service default/svc-nodeport-httpd in default browser...
```

## Deleting objects

You can delete objects individually, or collectively:

```bash
$ kubectl delete -f ./configs
pod "pod-httpd" deleted
service "svc-nodeport-apache-webserver" deleted
```

This might take a minute or 2 to complete, but you can speed it by setting the grace-period to something short, e.g. 2 seconds:

```bash
kubectl delete -f ./configs --grace-period=2
```

If you want to delete everything, you can do:

```bash
kubectl delete all --all
```

## More about Pods

This is just a quick intro to pods. We'll cover more about pods as we work through the rest of this course.

## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)