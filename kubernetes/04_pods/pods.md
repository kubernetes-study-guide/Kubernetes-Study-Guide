# Kubernetes Hello World


## Hello world - Part 1
In this walkthrough we will get an Apache web server (container) running inside our kube cluster. In Kubernetes we build objects. There are different types (aka kind) of objects, Pods, Services, Deployments,....etc. In our hello-world example we'll start by building a Pod object. 

**Pod:** A pod main purpose is to house one or more containers. In kubernetes you can only run container inside pods. That makes pods the fundamental building block in Kubernetes. At the moment, we don't have any pods in our kubecluster:

```bash
$ kubectl get pods
No resources found.
```

You create an object by first writing a yaml config file, and then feeding that config file into the kubectl command. So here's our pod's yaml content:

```yaml
---
apiVersion: v1      # see https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/
kind: Pod           # type of object that's defined in this file
metadata:
  name: pod-httpd # name of the pod to be created. 
  labels:
    component: apache_webserver  # this tag is added to help this object to link to the service object.
spec:
  containers:
    - name: cntr-httpd  # name of the container that will reside in the pod
      image: httpd    # using the official apache image from docker hub
      ports:
        - containerPort: 80  # what port the container will be listening on
```

We'll cover how to construct these yaml files from scratch in the anatomy tutorial later on. For now all you need to know is that this yaml file will instruct kubectl to:

1. create a pod object that consists of a single container.
2. Build this container using the official docker hub [httpd](https://hub.docker.com/_/httpd) image.
3. name the container 'cntr-httpd',
4. and name the pod itself, 'pod-httpd'.
5. note that the container will be listening on port 80
6. Assign an arbitrary key/value label to the pod of key=component and value=apache_webserver. This label will come in useful later on.

We wrote this yaml content into a file called, pod-httpd-descriptor.yml. It doesn't matter what you call the file, as long as it's meaningful to you. and ends with the .yml/.yaml suffix. Now let's create the object, using the apply command:


```bash
$ kubectl apply -f configs/pod-httpd-obj-def.yml
pod "pod-httpd" created

$ kubectl get -o wide pods
NAME        READY     STATUS    RESTARTS   AGE       IP           NODE
pod-httpd   1/1       Running   0          8s        172.17.0.5   minikube
```

Here we can see tha the new pod has been created. The pod's name is 'pod-httpd' which is exactly the name we asked for in our yaml file's metadata.name setting. Also pod has it's very own private IP auto-assigned to it. To get more detailed info about our pod, we can use the 'describe' subcommand:

```bash
kubectl describe pods pod-httpd
Name:               pod-httpd
Namespace:          default
Priority:           0
PriorityClassName:  <none>
.
.
...etc
```

So far we've created a single-container pod. This container is supposed to have the apache webserver running inside it. But how to do we verify that container's web service is definitely working? To properly verify this, we need to do a 2-step verification process:

1. Verify that our container (and consequently our pod) is listening on the correct port.
```bash
nc -v pod-ip 80
```
2. Try and access our containers homepage, e.g. using curl:
```bash
curl http://pod-ip
```

If you open up a bash terminal on your macbook and tried these nc and curl commands you'll find that they fail. That's because Kubernetes only comes with some basic networking features out-of-the-box. Those networking features only makes pods accessible by either the kube cluster's worker/master nodes, and from other pods in the kubecluster. This means you have to setup extra configurations to make pods externally accessible.


In the meantime there are some more limited tests that you can still perform, that is that you still perform the nc+curl tests but from inside the container itself. To do that, you need to access your container's bash terminal. You can do that by using the exec command:

```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
root@pod-httpd:/usr/local/apache2#
```

This command is quite similar to the docker command. In your case, you might need to replace '/bin/bash' with something else,e.g. '/bin/sh', bash, sh, depending on the image your container was created from. The '--' is used to tell kubectl that everything after '--' should be interpreted as the command to run inside the pod, which in our example is just to create a bash session.

Once you're inside the container, you then need to install the nc and curl packages. The command you need to run various depending on the image you use, but in our case, we run:

```bash
apt-get update
apt-get install netcat
apt-get install curl
```

Now we run the verification tests:

```bash
# nc -v localhost 80
localhost [127.0.0.1] 80 (?) open


root@pod-httpd:/usr/local/apache2# curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

Success!


You can also use 'kubectl exec' to directly run commands without needing to first creating a bash session inside the container:

```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- nc -v localhost 80
localhost [127.0.0.1] 80 (?) open
```

In this example if we did omit the '--', then kubectl would have because because kubectl would have thought that that '-v' is a kubectl flag rather than a netcat flag.


### Validate from inside the kube master/worker node


You can run the telnet+curl command from inside the minikube vm:

```bash
$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-httpd   1/1     Running   0          28m   172.17.0.7   minikube   <none>           <none>

$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl http://172.17.0.7
<html><body><h1>It works!</h1></body></html>
$ 
```


## Hello World - Part 2

We're now going to improve this hello-world example by making our pod accessible directly from our macbook's web browser. That's done by creating a 'service' object.

**Service:** A service object is used to setup networking in our kube cluster. E.g. if a running pod exposes a web based gui, then a service object needs to be set up to make that pod's gui externally accessible. 

So in our hello-world example, we've created a new yaml file with the content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport-apache-webserver
spec:
  type: NodePort   # there are 4 types of Services. ClusterIP, NodePort, LoadBalancer, Ingress.
                   # https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
                   # NodePort should only be used for dev environments.
  ports:
    - port: 3050  # this is used by other pods to access assets that's available in our demo conainer

      targetPort: 80 # port number of the pod's primary container is listening on. So
                       # needs to mirror containerPort setting as defined in the object config file.

      nodePort: 31000  # this ranges between 30000-32767. Our worker node VM will be listening on this port.
                       # It's actually the kube-proxy compoenent on worker nodes that will start listening on this port.
                       # this is the port number we need to enter into our web browser. That's one of the drawbacks
                       # in using nodePort service type, i.e. have to explicitly specify ugly port numbers in the url
  selector:
    component: apache_webserver  # this says it will forward traffic to object that has metadata.label entry
                                 # with key/value pair of 'component: web'
                                 # that's how this object and the pod object links together.
```

There are different types of service objects, in our case we are creating a NodePort type service. NodePort services are quite crude and isn't recommended for production, but we're using it here because it's the easiest service type to understand for a beginner. Before we create our service object, let's first see what services we currently have:

```bash
$ kubectl get services
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   10.96.0.1    <none>        443/TCP   3h
```

The 'kubernetes' service comes as default in an Kubernete install and is used for internal purposes only. Therefore you can ignore this service. Now let's create the service object:

```bash
$ kubectl apply -f configs/svc-nodeport-obj-def.yml
service "svc-nodeport-apache-webserver" created

$ kubectl get -o wide services
NAME                            CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE       SELECTOR
kubernetes                      10.96.0.1       <none>        443/TCP          4h        <none>
svc-nodeport-apache-webserver   10.100.173.40   <nodes>       3050:31000/TCP   7s        component=apache_webserver
```

**Handy Tip**: Notice that we had to run the apply command twice so far, once for each yaml file. Luckily there's a way to apply all the configs in one command by simply specifying the directory that houses all your configs, e.g.:

```bash
$ kubectl apply -f ./configs
pod "pod-httpd" created
service "svc-nodeport-apache-webserver" created
```

Next, you need to find the ip address of your worker node, which you can find by running:

```bash
$ minikube ip
192.168.99.100
```

Now you know the ip number and port number you should be using, So you can test the endpoint either via a web browser, or with curl:

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

Or to open it up in a web browser, just do:

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




## Troubleshooting pods

If a pod is failing to enter running mode, then there's a few ways to investigate that:


```bash

kubectl logs podname

kubectl logs podname --previous   # view previous crashed pod's log. 

kubectl describe pods podname   # this has a history session, which could give more info

kubectl get pods podname -o yaml   # this has a state message which gives more info too.

kubectl get events    # this give more general historical info about tasks performed by kubernetes



```

## More about Pods

This is just a quick intro to pods. We'll cover more about pods as we work through the rest of this course. 

## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)