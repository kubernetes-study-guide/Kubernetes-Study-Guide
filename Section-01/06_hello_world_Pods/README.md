# Hello World - Pods

In this walkthrough we will get an Apache web server (container) running inside our kube cluster. In Kubernetes we build objects. There are different types (aka kind) of objects, Pods, Services, Deployments,....etc. In our hello-world example we'll start by building a Pod object.

**Pod:** A pod main purpose is to house one or more containers. In kubernetes you can only run container inside pods. That makes pods the fundamental building block in Kubernetes. At the moment, we don't have any pods in our kubecluster:

```bash
$ kubectl get pods
No resources found.
```

You create an object by first writing a yaml config file that defines the object, and then feeding that config file into the kubectl command.

From this point forward we'll refer to these files as yaml descriptors. We'll store these yaml descriptors in a folder called 'configs' which sits alongside the article that your currently viewing.

So here's our pod's yaml descriptor:

```yaml
---
apiVersion: v1      # see https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/
kind: Pod           # type of object that's defined in this file
metadata:
  name: pod-httpd   # the name displayed in the first column of 'kubectl get pods'
  labels:
    app: apache_webserver  # this tag is added to help this object to link to the service object.
spec:
  containers:
    - name: cntr-httpd      # name of the container that will reside in the pod
      image: httpd:latest      # using the official apache image from docker hub, along with a tag
      ports:                # this bit is purely for informational purposes only and can be omitted.
        - containerPort: 80   # what port the container will be listening on

```

We'll cover how to construct these yaml files from scratch in the anatomy tutorial later on. For now all you need to know is that this yaml file will instruct kubectl to:

1. create a pod object that consists of a single container.
2. Build this container using the official docker hub [httpd](https://hub.docker.com/_/httpd) image.
3. name the container 'cntr-httpd',
4. and name the pod itself, 'pod-httpd'.
5. note that the container will be listening on port 80
6. Assign an arbitrary key/value label to the pod of key=app and value=apache_webserver. This label will come in useful later on.

We wrote this yaml content into a file called, pod-httpd-descriptor.yml. It doesn't matter what you call the file, as long as it's meaningful to you. and ends with the .yml/.yaml suffix. Now let's create the object, using the apply command:

```bash
$ kubectl apply -f configs/pod-httpd-obj-def.yml
pod "pod-httpd" created


$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-httpd   1/1     Running   0          90s   172.17.0.8   minikube   <none>           <none>


$ kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          114s   172.17.0.8   minikube   <none>           <none>            app=apache_webserver
```

Here we can see tha the new pod has been created. The pod's name is 'pod-httpd' which is exactly the name we asked for in our pod descriptor's `pod.metadata.name` setting. The pod has it's very own private IP auto-assigned to it. To get more detailed info about our pod, we use the yaml setting for the output flag:

```bash
$ kubectl get pods pod-httpd -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-05-14T19:36:33Z"
  labels:
    app: apache_webserver
  name: pod-httpd
  namespace: default
```

You can also use the 'describe' command to get more info too:

```bash
$ kubectl describe pods pod-httpd
Name:               pod-httpd
Namespace:          default
Priority:           0
PriorityClassName:  <none>
...
```


By the way, Whenever you come across a command where you specify an object type following by an objects name, you can actually insert a forward slash in the command as well, e.g.:

```bash
kubectl describe pods/pod-httpd
```

This is just another way of writing the same command. 

## Sanity check our pod

So far we've created a single-container pod. This container is supposed to have the apache webserver running inside it. But how to do we verify that container's web service is definitely working? E.g. we could do a 2-step verification process:

1.verify that our container (and consequently our pod) is listening on the correct port.

```bash
nc -v pod-ip 80

```

1.Try and access our containers homepage, e.g. using curl:

```bash
curl http://pod-ip
```

If you open up a bash terminal on your macbook and tried these nc and curl commands you'll find that they fail. That's because the pod's ip address is part of a network that's internal to the kube cluster.

Kubernetes only comes with some basic networking features out-of-the-box. Those networking features only makes pods accessible by:

- Internally by logging into the primary container itself, and then use localhost
- Internally by logging into the pods secondary container, if any
- kube cluster's worker/master nodes
- other pods in the same kubecluster

We'll cover how to set up external networkng later on. For now, we'll perform our checks using the first option, i.e. from inside the pod's container. To do that, you need to access your container's bash terminal. You can do that by using the exec command:

```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
root@pod-httpd:/usr/local/apache2#
```

This command is quite similar to the docker command. In our case, you might need to replace '/bin/bash' with something else, e.g. '/bin/sh', bash, sh, depending on what image your container was created from. The '--' is used to tell kubectl that everything after '--' should be interpreted as the command to run inside the pod, which in our example is just to create a bash session.


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

The '--' is used to tell kubectl that everythign after '--' is a command to run inside the container. In this example if we omitted the '--', then kubectl would have failed because it would have thought that that '-v' flag is a kubectl flag rather than nc's flag. However you can omit the '--' as long as the exec command doesn't contain any flags of its.

## Validate from inside the kube master/worker node

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
```


## Troubleshooting pods cheatlist

If a pod is failing to enter running mode, then there's a few ways to investigate that:

```bash

kubectl logs podname -c container_name

kubectl logs podname  -c container_name --previous   # view previous crashed pod's log.

kubectl describe pods podname   # this has a history session, which could give more info

kubectl get pods podname -o yaml   # this has a state message which gives more info too.

kubectl edit pods podname  # see yaml descriptor of pod

kubectl get events    # this give more general historical info about tasks performed by kubernetes

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