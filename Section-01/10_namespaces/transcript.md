# transcript

When you create a brand new kube cluster, and then list out your pods, you'll find there aren't any pods yet:

```bash
$ kubectl get pods
No resources found.
```

And consequently that means that we shouldn't have any containers running either. Now If your kubecluster is using docker, then you can run docker commands against your kubecluster's worker nodes. minikube happens to use docker as it's container run time engine, and it even comes with the docker-env helper command: 

```bash
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/schowdhury/.minikube/certs"
export DOCKER_API_VERSION="1.35"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

This command prints out a set of export commands that you need to run in order to get your workstation's docker cli to connect to the minikube VM's docker daemon. However the last line of this output shows how to activate these settings:


```bash
eval $(minikube docker-env)
```

Ok we're now connected to minikube's docker daemon for the rest of this bash session. So let's try listing our containers:

```bash
$ docker container ls
```

Ok so based on what we know so far, we should have got an empty list, since we don't have any pods. So where have all these containers come from?

The answer is to do with namespaces. You see, in Kubernetes we can group our objects into a construct known as **namespaces**. We can list all the namespaces using the get command:

```bash
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    11h
kube-public   Active    11h
kube-system   Active    11h
```

our minikube cluster comes with these namespaces already setup. Kubectl can only interact with one namespace at a time. We can run the get-context config command to see which namespace our kubectl client is currently pointing to:

```bash
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube
```

This command shows some of the info stored in the kubectl config file. By default this file is called config,it's located in your home directory.


```popup animation
~/.kube/config
```

We'll cover more about this file later in the course.

If the current context shows as blank, like it is here, then it means that we're using the default namespace [red box]. So when we ran the get-pods command we only saw a list of pods that are residing in the default namespace.


If you want to interact with a different namespace, then you can do so by specifying the namespace flag:


```bash
$ kubectl get pods --namespace=kube-system
```

Kubernetes is using this namespace to house objects that it itself needs for it's own internal working. Here we see that we have a lot of pods, which explains where all those containers came from earlier.

You can also create your own namespaces and organise your objects into them. Here's a yaml file to create a namespace called dev1:

```bash
$ tree configs/
$ code configs/namespace-dev1.yml
```

In my case I called my namespace ns-dev1 becuase I want to have several dev environments and dev1 will store all the objects for the first development environment. You can name your namespaces to whatever makes sense in your workplace, such as team names, project names, environment names, 

Note that this is one of the few object types where you don't need to explicitly add a spec section. Lets now create this namespace:

```bash
kubectl apply -f configs/namespace-dev1.yml
kubectl get namespaces
```

Next you'll want to add objects to your namespace. You can do this by using the namespace flag:

```bash
kubectl apply -f configs/pod-httpd.yml --namespace ns-dev1   # just show but dont run. 
```

This is a great way to deploy the same pod across several namespaces. All you have to do is run the above command multiple times with a different namespace each time.

However if you have an object that's only supposed to be deployed to a particular namespace, then you can set this in the yaml file instead:

```bash
code configs/pod-httpd.yml 
```

This is the same file as we used in the hello world pod demo, but this time with a metadata.namespace setting added in. In this case you don't need to use the namespace flag anymore:

```bash
kubectl apply -f configs/pod-httpd.yml
```

We can doublecheck which namespace an object belongs to using the get command:

```bash
kubectl get pods pod-httpd -o yaml --namespace=ns-dev1 | head 
```

The fact that we used the --namespace flag alone is proof enough. but you can view this output just to be extra sure. 



Now if you want to work with a particular namespace for a prolonged period then writing out the namespace flag all the time can get tedious. Instead you can persistently change the default namespace:

```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=ns-dev1
$ kubectl config get-contexts
```

This now shows our chosen namespace. All that's happened behind the scenes to make this change persistent, is a single line was changed in kubectl's config file.




Some objects are cluster level objects and can't be namespaced. You can use the api-resources command to get a list of which objects can be namespaced and which cant:

```bash
$ kubectl api-resources -o wide
```

That's it for this video. See you in the next one. 

