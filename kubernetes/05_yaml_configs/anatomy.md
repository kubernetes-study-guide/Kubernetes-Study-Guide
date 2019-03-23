# Anatomy of an Kubernetes object config file

There's 3 main approaches to create/update/delete Kubernetes objects. Broadly speaking, they are the [imperative and declarative approaches](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/). 

Broadly speaking, the imperative approach involves creatinng objects solely via the kubectl command line. Whereas the declaritive approach involves createing 'Kubernetes Object yaml files'. Then we use the kubectl 'apply' subcommand to create these objects. 

The Declaritive approach is deemed as the best practice approach. 

 Notice, that all these config files have the following general yaml structure:

```yaml
apiVersion: xxx
kind: xxxxx
metadata:
  {blah blah blah}
spec:
  {blah blah blah}
```

The apiVersion, kind, metadata, and spec, are the [required fields](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields), for all kubernetes object files.

**kind:** What type object that you want to make.

**apiVersion:** The kubernetes api is rapidly evolving so the api is broken down into various parts. Your version choice depends on what 'kind' of object you want to define.  For example, if the kind is 'Pod' then this field needs to be set to 'v1'.

You need to take a look at the [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/) to work out what to set this for your chosen object type (kind). This reference doc is really useful have displays example yaml content for your chosen kind.
This link is for version v1.13. But in your case you need go to the link, that's specified with the Major and Minor tag of Server Version in:

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.3", GitCommit:"721bfa751924da8d1680787490c54b9179b1fed0", GitTreeState:"clean", BuildDate:"2019-02-04T04:48:03Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.3", GitCommit:"721bfa751924da8d1680787490c54b9179b1fed0", GitTreeState:"clean", BuildDate:"2019-02-01T20:00:57Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```

You can also view the api reference data from the cli, using the 'explain' subcommand:

```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.
.
.
...etc
```

And you can drill down like this:


```bash
$ kubectl explain pod.kind
KIND:     Pod
VERSION:  v1

FIELD:    kind <string>

DESCRIPTION:
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds
```


**metadata:** Data that helps uniquely identify the object. metadata.name is used to assign a name to the object. metadata.labels is also another really important feature. It not only lets you organise your resources, but it also offers a means to filter your resources, and can be used as a way to refer to a group of resources. 

**spec:** The content of this depends on the kind of object in question. Api specifies what structure+content this section should hold.

## Start creating the Kubernetes objects

First let's get the current status:

```bash
$ kubectl get pods
No resources found.
$ kubectl get services
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   10.96.0.1    <none>        443/TCP   3h
```

Notice, we already have a service called 'kubernetes'. This came as default and is used by Kubernete for internal purposes only. Therefore you can ignore this service.

Now let's create the objects, starting with the pod object:

```bash
$ kubectl apply -f configs/pod-httpd-obj-def.yml
pod "pod-httpd" created

$ kubectl get -o wide pods
NAME        READY     STATUS    RESTARTS   AGE       IP           NODE
pod-httpd   1/1       Running   0          8s        172.17.0.5   minikube
```

Notice here that we used the '-o wide' just to get some verbose info. Next let's do the service object:

```bash
$ kubectl apply -f configs/svc-nodeport-obj-def.yml
service "svc-nodeport-apache-webserver" created

$ kubectl get -o wide services
NAME                            CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE       SELECTOR
kubernetes                      10.96.0.1       <none>        443/TCP          4h        <none>
svc-nodeport-apache-webserver   10.100.173.40   <nodes>       3050:31000/TCP   7s        component=apache_webserver
```

Another way to get more details info is by using the 'describe' subcommand:

```bash
kubectl describe pods pod-httpd
Name:               pod-httpd
Namespace:          default
Priority:           0
PriorityClassName:  <none>
.
.
...etc 


kubectl describe service svc-nodeport-apache-webserver
Name:                     svc-nodeport-apache-webserver
Namespace:                default
Labels:                   <none>
.
.
...etc
```





Notice that we had to run the apply command twice, once for each config file. Luckily there's a way to apply all the configs in one command, and that is to just specify the directory that houses all the configs, e.g.:

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

Now, you can test either via a web browser, or with curl:

```bash
$ curl http://192.168.99.100:31000
<html><body><h1>It works!</h1></body></html>
```

## Deleting objects

You can delete objects individually, or collectively. Here's how to do it collectively:

```bash
$ kubectl delete -f ./configs
pod "pod-httpd" deleted
service "svc-nodeport-apache-webserver" deleted
```

## Defining multiple objects in a single config file

In this walkthrough we ended up with 2 config files. However you can store 2 or more objects in a single config file. All you need to do is to copy all the definitions into a single file, and seperate them out using by inserting the yaml-new-document-syntax '---' between them. It's really a preference on whether or not to use this approach.


## Updating objects

Kubernetes is smart enough to identify which objects have been created by a particular config file. It does so by using the configs about the 'kind' and metadata.name info. Config's filename itself doesn't matter, as long as it ends with the .yml/.yaml extension. You can make changes to the yaml files (as long as it isn't changing the kind or metadata.name fields) and just apply them again for the changes to take affect.

That's a good thing because it means you can modify an existing object by editing it's corresponding config file and reapply it. However, since the 'kind' and 'metadata.name' are used for mapping yaml configs to their respective objects it means that changing these will make kubernetes think that they are new objects. In my example I'm going to change the pod's image name from httpd to nginx:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: apache_webserver
spec: 
  containers:
    - name: cntr-httpd
      image: nginx       # this was httpd back when this object was created. 
      ports:
        - containerPort: 80

```

Now we reapply the changes:

```bash
$ kubectl apply -f configs/pod-object-definition.yml 
pod "pod-httpd" configured
```

This time we get a configured message. This means that kubernetes hasn't created anything new, just updated one or more existing objects. Now we can retest:

```bash
$ curl http://192.168.99.100:31000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

However, like 'kind' and metadata.name, there are other fields that you can't change, e.g. for a pod object, you can't change the containerPort. If you do then you'll get a forbidden error message.



## View entire yaml definition including implicit values. 



## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)