# Kubernetes Hello World

In our hello world example, we created a single pod. however when you want to create multiple identical pods. One way to do this is to create a ReplicaSet object.

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-httpd
  labels:
    component: httpd_webserver
spec:
  replicas: 2   # This sets the number of pods that needs to exist.
                # This is a bit like the ASG equivalent of min/desired/max, but as a single value.
  selector:
    matchLabels:
      component: httpd_webserver  # this is used by the ReplicaSet object to keep track
                                  # of which pods it's configuration created.

  template:  # the content nested in this section is pretty much what was in the pod-definition yaml file.
             # I.e. it's the aws equivalent of the EC2 Launch Configuration.
    metadata:
      labels:
        component: httpd_webserver  # this needs to be match up with the matchLabels setting above,
                                    # so that the replicaset knows which pods it has created.
    spec:
      containers:
        - name: cntr-httpd
          image: httpd:latest
          ports:
            - containerPort: 80
          command: ["/bin/bash", "-c"]     # We have performed a command override here for testing purposes.
          args:
            - |
              /bin/echo "The pod, $HOSTNAME is displaying this page." > /usr/local/apache2/htdocs/index.html
              /usr/local/bin/httpd-foreground
```

Notice here that we have done a 'command override'. The official [httpd docker image](https://github.com/docker-library/httpd) comes built in with a command to run during the container's launchtime, this default command is `/usr/local/bin/httpd-foreground`. However we wanted to create a custom index.html file to showcase how replicasets work in this demo. We've therefore opted to create the index.html file by running a 'echo' command using the pod.spec.containers.command and pod.spec.containers.args settings. This ends up overriding the default command, which is not what we wanted. So to counter this problem, we've appended `/usr/local/bin/httpd-foreground` to the end of our args setting, so that it still gets executed during the container's launch time. This is a bit of a hacky approach, there are better ways to do this, such as using configmaps, volumes, and postStart hooks. We'll cover these approaches later.

This creates the replicaset:

```bash
$ kubectl get replicasets
NAME       DESIRED   CURRENT   READY   AGE
rs-httpd   2         2         2       32s
```

This replicaset in turn creates the following pods:

```bash
$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
rs-httpd-7rnj8   1/1     Running   0          101s
rs-httpd-grm6m   1/1     Running   0          101s
$
```

As you can see, a single yaml file ended up creating multiple objects. 

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

## Use Deployments instead of ReplcaSets

In reality [you will never need to create ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#when-to-use-a-replicaset), you would create Deployment objects instead.






