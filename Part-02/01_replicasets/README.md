# Replicasets

In our hello world example, we created a single pod. however when you want to create multiple identical pods. One way to do this is to create a ReplicaSet object.

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-httpd
spec:
  replicas: 2   # This sets the number of pods that needs to exist.
  selector:
    matchLabels:
      component: httpd_webserver  # The rs object manages all pods that has this label
  template:  # this is used to nest a pode descriptor inside this rs descriptor.
    metadata:
      labels:
        component: httpd_webserver  # this needs to be match with matchLabels above.
    spec:
      containers:
        - name: cntr-httpd
          image: httpd:latest
          ports:
            - containerPort: 80
          command: ["/bin/bash", "-c"]     # We have performed a command override here for testing purposes.
          args:
            - |
              /bin/echo "You've hit - $HOSTNAME" > /usr/local/apache2/htdocs/index.html
              /usr/local/bin/httpd-foreground  
```

Notice here that we have done a 'command override'. The official [httpd docker image](https://github.com/docker-library/httpd) comes built in with a command to run during the container's launchtime, this default command is `/usr/local/bin/httpd-foreground`. However we wanted to create a custom index.html file to showcase how replicasets work in this demo. We've therefore opted to create the index.html file by running a 'echo' command using the pod.spec.containers.command and pod.spec.containers.args settings. This ends up overriding the default command, which is not what we wanted. So to counter this problem, we've appended `/usr/local/bin/httpd-foreground` to the end of our args setting, so that it still gets executed during the container's launch time. This is a bit of a hacky approach, there are better ways to do this, such as using configmaps, volumes, and postStart hooks. We'll cover these approaches later.

This creates the replicaset:

```bash
$ kubectl get replicasets
NAME       DESIRED   CURRENT   READY   AGE
rs-httpd   2         2         2       32s
```

replicasets has an alias which is 'rs', so you can run the above as:

```bash
$ kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
rs-httpd   2         2         2       32s
```

For a list of all aliases, see:

```bash
$ kubectl api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
...
```

Also notice that this table species which objects are namespace level objects, and which are cluster-wide. This replicaset in turn creates the following pods:

```bash
$ kubectl get pods -o wide --show-labels
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
rs-httpd-mkg65   1/1     Running   0          12m   172.17.0.7   minikube   <none>           <none>            component=httpd_webserver
rs-httpd-sn5mt   1/1     Running   0          12m   172.17.0.8   minikube   <none>           <none>            component=httpd_webserver
```

As you can see, a single yaml file ended up creating multiple objects. Notice that the pods name inherits the replicaset's name followed by '-randomstring', in order for the pods to have unique name.  

We've also created a NodePort service that points to all the pods created by the replicaset.

```bash
$ kubectl get service -o wide
NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
svc-nodeport-httpd-webserver   NodePort   10.110.27.161   <none>        3050:31000/TCP   10m   component=httpd_webserver
$ kubectl describe svc svc-nodeport-httpd-webserver | grep Endpoint
Endpoints:                172.17.0.7:80,172.17.0.8:80

$ kubectl get endpoints svc-nodeport-httpd-webserver
NAME                           ENDPOINTS                     AGE
svc-nodeport-httpd-webserver   172.17.0.2:80,172.17.0.8:80   10m
```

The nodePort service isn't aware that the replicaset created the pods. All it's interested in is forwarding traffic to pods with labels that matches the Nodeport object's selector:

```bash
$ curl http://$(minikube ip):31000
You've hit - rs-httpd-mkg65
$ curl http://$(minikube ip):31000
You've hit - rs-httpd-sn5mt
$ curl http://$(minikube ip):31000
You've hit - rs-httpd-mkg65
$ curl http://$(minikube ip):31000
You've hit - rs-httpd-sn5mt
```

## matchExpressions

In the above example we used 'matchLabels', however as an alternative you can use `rs.spec.selector.matchExpressions` which is more flexible, than matchLabels.

## Controllers

The word 'controller' is used in Kubernetes to refer to objects that doesn't actually to any heavy-lifting low level work, e.g. it doesn't run any containers like pods do, or route traffic, like nodeport service objects do. Instead a controller object creates other objects that does all the legwork, and it ensures the state of those objects. Therefore a controller builds other objects to reach a desired state, and then constantly monitors+maintains that desired state.

Objects like replicasets are referred to as 'Controller objects'. A controller object is an object that controls the creation/deletion of other objects, and monitors the health of those objects. So in the case of replicaset objects, Replicasets creates/deletes/maintains pod objects, as per the spec.replicas setting. For example if we have 2 pods created/monitored by a replicaset,and we simulate a pod failure by deleting one of them, then the replicaset will automatically build another pod in order to reach the desired state:

```bash
$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
rs-httpd-k62st   1/1     Running   0          96s
rs-httpd-rtgtf   1/1     Running   0          96s

$ kubectl delete pod rs-httpd-rtgtf
pod "rs-httpd-rtgtf" deleted

$ get pods
NAME             READY   STATUS    RESTARTS   AGE
rs-httpd-9nkwm   1/1     Running   0          5s     # new pod created.
rs-httpd-k62st   1/1     Running   0          118s
```

However if you delete the RS, then you end up deleting the objects the RS controls:

```bash
$ kubectl delete rs rs-httpd
replicaset.extensions "rs-httpd" deleted
$ kubectl get pods
No resources found.
```

On the kube master there is a dedicated systemd service that runs the [kube-controller-manager](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/08-bootstrapping-kubernetes-controllers.md#configure-the-kubernetes-controller-manager). That component is responsible for looking after all the controller objects.

## Use Deployments instead of ReplcaSets

There are lots of other types of controller objects, e.g. Deployments, StatefulSets,...etc, we'll cover all them later. In reality [you will never need to create ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#when-to-use-a-replicaset), you would create Deployment objects instead.
