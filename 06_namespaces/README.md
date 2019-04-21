# namespaces

Earlier, when we tried to get a list of all pods, we got:

```bash
$ kubectl get pods
No resources found.
```

This is not strictly true. In Kubernetes we have a feature called namespaces that lets us segment/organise all our objects into a construct known as namespaces. Here's the current list of namespaces:

```bash
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    11h
kube-public   Active    11h
kube-system   Active    11h
```

These 3 namespaces comes included in a fresh kubernetes install.

When we run the `kubectl get pods` earlier, we didn't specify a namespace, so kubectl by default assumed we are only interested in the 'default' namspace. The 'kube-system' namespace store objects that are used by kubernetes internally. For example here's how to view what pods k8s uses internally:

```bash
$ kubectl get pods --namespace=kube-system
NAME                                   READY     STATUS    RESTARTS   AGE
coredns-86c58d9df4-cnqnr               1/1       Running   0          11h
coredns-86c58d9df4-ct8fx               1/1       Running   0          11h
etcd-minikube                          1/1       Running   0          11h
kube-addon-manager-minikube            1/1       Running   0          11h
kube-apiserver-minikube                1/1       Running   0          11h
kube-controller-manager-minikube       1/1       Running   0          11h
kube-proxy-8zwff                       1/1       Running   0          11h
kube-scheduler-minikube                1/1       Running   0          11h
kubernetes-dashboard-ccc79bfc9-l66xk   1/1       Running   0          11h
storage-provisioner                    1/1       Running   0          11h
```

Or to get all kube-system namespace objects, do:

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

You can organise your objects in various ways using namespace. For example, you can create namespaces called 'Prod' and 'Dev', although it's best to segment prod/dev by having seperate kubernetes cluster altogether. Alternatively, have namespaces based on project/programme name. Here's how to create a new namespace imperitively:

```bash
$ kubectl create namespace codingbee-hello-world
namespace "codingbee-hello-world" created
```

Alternatively we can create it via the yaml approach:

```bash
---
apiVersion: v1
kind: Namespace
metadata:
  name: codingbee-hello-world
spec: {}
```


You can then specify what namespace objects are created in by adding the following line in the metadata section:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  namespace: codingbee-hello-world       # we add this line.
  labels:
    app: apache_webserver
spec:
  containers:
    - name: cntr-httpd
      image: httpd
      ports:
        - containerPort: 80
```

And for the service we have:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport-apache-webserver
  namespace: codingbee-hello-world       # we add this line.
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
$ kubectl get all -o wide --namespace=codingbee-hello-world
NAME            READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
pod/pod-httpd   1/1     Running   0          107s   172.17.0.8   minikube   <none>           <none>

NAME                                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/svc-nodeport-apache-webserver   NodePort   10.107.98.250   <none>        3050:31000/TCP   62s   app=apache_webserver
```

## Cluster level objects

Not all kubernetes objects are 'namespaced', i.e. can be organised into a namespace, because they are cluster level objects.

```bash
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

## Persistently Setting the namesapce
Specifying namespaces on the command line can get quite tedious, which might put you off from using namespaces. However you can persistantly change namespaces by running:

```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=codingbee-hello-world
Context "minikube" modified.
```

The above command essentially makes a one-line change in your `~/.kube/config` file. 

```bash
$ diff ~/.kube/config ~/.kube/config-orig
26c26
<     namespace: codingbee-hello-world
---
>     namespace: kube-system


$ grep 'codingbee-hello-world' ~/.kube/config -A2 -B2
- context:
    cluster: minikube
    namespace: codingbee-hello-world
    user: minikube
  name: minikube
```

This results in:

```bash
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          default              kubernetes                   chowdhus
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
*         minikube             minikube                     minikube             codingbee-hello-world
```

This command essentially displays some of the the content extracted from `~/.kube/config` in a more readable form. As you can see, the command we use to configure which kubecluster our kubectl command should connect (i.e. the context) to also lets you set the namespace (i.e. namespace to use when not explicitly specified on the command line).



