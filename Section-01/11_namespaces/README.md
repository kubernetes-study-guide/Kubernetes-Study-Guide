# namespaces

When you create a brand new kube cluster, and then get a list of pods, you'll get:

```bash
$ kubectl get pods
No resources found.
```

This is not strictly true. In Kubernetes we have a feature called namespaces that lets us segment/organise all our objects into a construct known as **namespaces**. Here's the current list of namespaces:

```bash
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    11h
kube-public   Active    11h
kube-system   Active    11h
```

These 3 namespaces comes included in a fresh kubernetes install.

When we run the `kubectl get pods` earlier, we didn't specify a namespace, so kubectl by default will use the 'default' namspace. The 'kube-system' namespace store objects that are used by kubernetes internally. For example here's how to view what pods k8s uses internally:

```bash
$ kubectl get all --namespace=kube-system
NAME                                      READY     STATUS    RESTARTS   AGE
po/coredns-86c58d9df4-cnqnr               1/1       Running   0          11h
po/coredns-86c58d9df4-ct8fx               1/1       Running   0          11h
po/etcd-minikube                          1/1       Running   0          11h
po/kube-addon-manager-minikube            1/1       Running   0          11h
po/kube-apiserver-minikube                1/1       Running   0          11h
po/kube-controller-manager-minikube       1/1       Running   0          11h
po/kube-proxy-8zwff                       1/1       Running   0          11h
po/kube-scheduler-minikube                1/1       Running   0          11h
po/kubernetes-dashboard-ccc79bfc9-l66xk   1/1       Running   0          11h
po/storage-provisioner                    1/1       Running   0          11h

NAME                       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
svc/kube-dns               10.96.0.10      <none>        53/UDP,53/TCP   11h
svc/kubernetes-dashboard   10.103.63.176   <none>        80/TCP          11h

NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/coredns                2         2         2            2           11h
deploy/kubernetes-dashboard   1         1         1            1           11h

NAME                                DESIRED   CURRENT   READY     AGE
rs/coredns-86c58d9df4               2         2         2         11h
rs/kubernetes-dashboard-ccc79bfc9   1         1         1         11h
```

Namespaces lets you can organise your objects in various ways. For example, we can create namespaces called 'Prod' and 'Dev'. Here's a yaml example for a namespace:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: ns-dev
```

> Notice that we didn't need to specify a namespace.spec section.



After creating the namespace, we can view it like this:

```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   5h19m
kube-node-lease   Active   5h19m
kube-public       Active   5h19m
kube-system       Active   5h19m
```

You can then create objects in your new namespace by using the `kubectl apply --namespace -f ...`. However I prefer the declaritive approach, by specifing what namespace objects belong to using the `xxx.metadata.namespace` setting:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  namespace: ns-dev       # we add this line.
  labels:
    app: apache_webserver
spec:
  containers:
    - name: cntr-httpd
      image: httpd
      ports:
        - containerPort: 80
```

And for our service we have:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport-apache
  namespace: ns-dev       # we add this line.
spec:
  type: NodePort
  ports:
    - port: 3050
      targetPort: 80
      nodePort: 31000
  selector:
    app: apache_webserver
```

We can apply them using the usual apply commands. And then we can check that they have been created by running:

```bash
$ kubectl get pods
No resources found.

$ kubectl get all -o wide --namespace=ns-dev
NAME            READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
pod/pod-httpd   1/1     Running   0          107s   172.17.0.8   minikube   <none>           <none>

NAME                                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/svc-nodeport-apache-webserver   NodePort   10.107.98.250   <none>        3050:31000/TCP   62s   app=apache_webserver
```

## Cluster level objects

Some objects are cluster level objects and therefore can't be namespaced (i.e. organised into namespaces):

```bash
$ kubectl api-resources -o wide
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND                             VERBS
bindings                                                                      true         Binding                          [create]
componentstatuses                 cs                                          false        ComponentStatus                  [get list]
configmaps                        cm                                          true         ConfigMap                        [create delete deletecollection get list patch update watch]
endpoints                         ep                                          true         Endpoints                        [create delete deletecollection get list patch update watch]
events                            ev                                          true         Event                            [create delete deletecollection get list patch update watch]
limitranges                       limits                                      true         LimitRange                       [create delete deletecollection get list patch update watch]
namespaces                        ns                                          false        Namespace                        [create delete get list patch update watch]
nodes                             no                                          false        Node                             [create delete deletecollection get list patch update watch]
...
```

## Set the namespace persistently

If you want to perform a lot of tasks against a particular namespace, then specifying namespaces on the command line every time can get quite tedious. However you can persistantly change namespaces by running:

```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=ns-dev
Context "minikube" modified.
```

This command is setting the user+cluster details via the current-context setting, and the sets the namespace to go with it. This command essentially makes a one-line change in your `~/.kube/config` file.

```bash
$ diff ~/.kube/config ~/.kube/config-orig
26c26
<     namespace: ns-dev
---
>     namespace: kube-system


$ grep 'ns-dev' ~/.kube/config -A2 -B2
- context:
    cluster: minikube
    namespace: ns-dev
    user: minikube
  name: minikube
```

This means the context called 'minikube' is telling kubectl which kubecluster to connect to, as which user, and which namespace to use as the default namespace (if the namespace flag isn't specified on the command line).

This results in:

```bash
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          default              kubernetes                   chowdhus
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
*         minikube             minikube                     minikube             ns-dev
```

This command essentially displays some of the the content extracted from `~/.kube/config` in a more readable form.