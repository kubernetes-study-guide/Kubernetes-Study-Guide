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
  name: ns-dev1
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
  namespace: ns-dev1       # we add this line.
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
  namespace: ns-dev1       # we add this line.
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
$ kubectl config set-context $(kubectl config current-context) --namespace=ns-dev1
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


## Using Docker with Minikube

The kubecluster running inside the minikube vm actually uses Docker to run all the containers. So when you create kubernetes objects, e.g. pods, then you can use the docker cli to view the underlying containers that have been created. You might want to do this for troubleshooting/debugging purposes.

In our macbooks, the docker cli is preconfigured to interact with the docker daemon that's running directly on our macbook. At the moment our macbook isn't directly running any containers:

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

However to interact with the minikube's docker daemon, you need to configure your macbook's docker cli to connect to your minikube's docker daemon. That's done by simply setting some environment variables, DOCKER_HOST, DOCKER_CERT_PATH,...etc. The minikube cli helpfully provides these environment variables:

```bash
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/schowdhury/.minikube/certs"
export DOCKER_API_VERSION="1.35"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

This output also give you a one-liner setup:

```bash
eval $(minikube docker-env)
```

This will only make the changes for the current session and will get reset when you restart are your bash terminal (to make this change permenant you need to add this into your bash profile script files). You should now see something like:

```bash
$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes                                                                             k8s_default-http-backend_default-http-backend-5ff9d456ff-m62k8_kube-system_cee7bc7a-4001-11e9-9566-080027d15c4c_0
da459f1cfb48        quay.io/kubernetes-ingress-controller/nginx-ingress-controller   "/entrypoint.sh /ngi…"   2 minutes ago       Up 2 minutes                                                                             k8s_nginx-ingress-controller_nginx-ingress-controller-7c66d668b-xq5gj_kube-system_cf8d09f1-4001-11e9-9566-080027d15c4c_0
...
```

Notice that we already have some containers running, that's because these containers are used by kubernetes itself for it's internal workings. If you don't have docker installed on your macbook, then you can still use the docker cli that's preinstalled inside the minikube vm. You do that by first ssh'ing into the minikube VM:

```bash
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)




$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes
...
```

### References

[https://kubernetes.io/docs/setup/minikube/](https://kubernetes.io/docs/setup/minikube/)
[https://kubernetes.io/docs/tutorials/hello-minikube/](https://kubernetes.io/docs/tutorials/hello-minikube/)
[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/tutorials/hello-minikube/)  (talks about getting autocomplete to work)
