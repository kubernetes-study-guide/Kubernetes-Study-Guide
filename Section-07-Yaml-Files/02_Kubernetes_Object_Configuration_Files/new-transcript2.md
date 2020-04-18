# Step yaml definition from scratch

You create a yaml definition by first creating an empty file. It doesn't matter what you call it, as long it ends with with .yml or .yaml. 

```bash
touch testfile.yml
code testfile.yml
```

All yaml definition have three top level keys, they are:

```type
apiVersion:
metadata:
kind:
ctrl+~
```

apiVersion, metadata, and kind. 

Before you can start filling out the rest, you first need to decide on what kind of object you want to define. You can use the api-resource command to get a list of the available object types:


```bash
cmd-   # make screen smaller
kubectl api-resource
then scroll up
```

As you can see we have a lot of different kinds of objects, and we have only touched on 2 kinds so far, pods and services. We'll cover more of these kinds as we go through the course. 

But for this demo we'll write a yaml definition for a pod. So let's declare that in our yaml file, by setting the kind as 'pod'.

By the way, notice the shortname column. You can use these aliases to cut down on typing. For example to you can run the get services command using the svc alias:


```bash
$ kubectl get svc -o wide
```



Next we have to set the apiVersion. The 'apiVersion' sets which part of the kubernetes api to access. This depends on the object's kind, which is why we had to set the kind, before assinging a value to this key.  This is a string value, and the explain command will tell you what to set here:

```bash
kubectl explain pod
```

It's basically the version setting. 

```type
type v1 in file
```

Next we have metadata, This is mainly used for storing info to help uniquely identify the object, such as the object's name. 

```bash
$ kubectl explain pod.metadata.name
```

For this demo, I'll name my pod, pod-httpd. 

```type
pod-httpd
```

Theres a lot of other things that can go under the metadata section, for example we can attach labels:

...


That's it, we've now created the mandatory parts of our yaml definition. However there can be additional object type specific sections that must be defined. For example, for pod objects, a spec section is required:


```type
spec:
```

For pods there are a lot of things that can be added in here, and we'll cover some of these as we go through the course. But for now, we'll add the containers section, followed by our first container definition. 


You don't need to explicitly specify the port details, but it's good practice to put it in, becuase these yaml definitions act as documentation as well as code. 









