# YAML files

In our hello-world demo we created a pod and service object by feeding yaml files into kubectl. But what is yaml?

Yaml is just a markup language like xml or json. The Yaml syntax is used for writing data in a structured way. I recommend taking look at the [Ansible website's yaml guide](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) if you want to learn about the syntax.

Basically YAML is based on a key-value system. Where the key is a string, and the value is essentially a container that can hold all kinds of things, such as:

- A string
- An array of values (which in turn are containers)
- Another key value pair
- A dictionary (a set of key value pairs).

One of the nice things about yaml is that it's relatively human readable when compared to xml or json. 

**Watch Out**: Yaml files are space sensitive. So need to always make sure all your indents are correct.

The yaml files we created for Kubernetes had the the following general yaml structure:

```yaml
apiVersion: xxx
kind: xxxxx
metadata:
  ...
spec:
  ...
```

**Watch out**: key names (e.g. apiVersion) are all case sensitive


The apiVersion, kind, metadata, and spec, are the [required fields](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields), for all kubernetes object files.

**kind:** What type of object that you want to create.

**apiVersion:** The kubernetes api is rapidly evolving so the api is broken down into various parts. Your version choice depends on what 'kind' of object you want to define.  For example, if the kind is 'Pod' then this field needs to be set to 'v1'.

**metadata:** Data that helps uniquely identify the object. metadata.name is used to assign a name to the object. metadata.labels is also another really important feature. It not only lets you organise your resources, but it also offers a means to filter your resources, and can be used as a way to refer to a group of resources.

**spec:** The content of this depends on the kind of object in question. Api specifies what structure+content this section should hold.


The first three items are mandatory. In most cases spec is mandatory as well. But that really depends on the kind of object you are creating. for example when create namespaces. you can leave out the spec setting. 

## Defining multiple objects in a single config file

In this walkthrough we ended up with 2 config files. However you can store 2 or more objects in a single config file. All you need to do is to copy all the definitions into a single file, and seperate them out using by inserting the yaml-new-document-syntax '---' between them. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: apache_webserver
spec:
  containers:
  - name: cntr-httpd
    image: httpd:latest
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport-httpd
spec:
  type: NodePort
  ports:
  - port: 3050
    targetPort: 80
    nodePort: 31000
  selector:
    app: apache_webserver

```

It's really a preference on whether or not to use this approach.


## Updating objects

Kubernetes is smart enough to identify which objects have been created by a particular config file. It does so by using the configs about the 'kind' and metadata.name info. The yaml descriptor's filename itself doesn't matter, as long as it ends with the .yml/.yaml extension. You can make changes to the yaml files (as long as it isn't changing the 'kind' or 'metadata.name' fields) and just apply them again for the changes to take affect.

## Tips of writing YAML files

When writing yaml files, You can refer to the [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/) to work out what to set for your chosen object. This reference doc is really useful and it displays example yaml code. This link is for version v1.14. But in your case you need go to the link, that's specified with the Major and Minor tag of Server Version in:

```bash
$ kubectl version --short
Client Version: v1.13.4
Server Version: v1.14.0
```

This shows that the kubectl binary we have installed locally on our macbook is v1.13.4, whereas the kubecluster our kubectl command is talking to is version v1.14.0.

You can also view the api reference data from the cli, using the 'explain' subcommand:

```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.
...
```

You can get a high-level overview of the entire yaml structure:

```bash
kubectl explain pod --recursive | less
```

And you can drill down like this:

```bash
$ kubectl explain pods.spec.containers.env --recursive
KIND:     Pod
VERSION:  v1

RESOURCE: env <[]Object>

DESCRIPTION:
     List of environment variables to set in the container. Cannot be updated.
...
```


## View entire yaml definition including implicit values

What if you have a pod, e.g. called pod-httpd, but lost it's yaml descriptor file? In that case you can regenerate the yaml descriptor from the existing pod:

```bash
kubectl get pod pod-httpd -o yaml > regenerated-descriptor.yaml
```

## Use Kubectl to generate YAML Boilerplate templates

One very handy feature with imperative commands is that you can use them to generate example yaml files. You can then use these as boilerplate templates to create your own yaml files:

```bash
kubectl run pod-httpd --image=httpd --restart=Never --dry-run -o yaml > pod.yaml

kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 --dry-run -o yaml > service.yaml
```

**CKA Exam tip:** Your not allowed to copy+paste more than a couple of lines at a time during the exam. You can write them by hand, but that's time consuming and prone to errors, which will use up even more precious seconds. Which is why it's best to use the imperative commands to create the yaml files for you. We'll include a complete list of these commands in the appendix. 

## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)

[https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/](https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/)