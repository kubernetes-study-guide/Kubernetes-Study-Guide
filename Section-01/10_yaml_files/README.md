# YAML files

In our hello-world demo we created a pod and service object by feeding yaml files into kubectl.

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
  {blah blah blah}
spec:
  {blah blah blah}
```

**Watch out**: key names (e.g. apiVersion) are all case sensitive


The apiVersion, kind, metadata, and spec, are the [required fields](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields), for all kubernetes object files.

**kind:** What type of object that you want to create.

**apiVersion:** The kubernetes api is rapidly evolving so the api is broken down into various parts. Your version choice depends on what 'kind' of object you want to define.  For example, if the kind is 'Pod' then this field needs to be set to 'v1'.

**metadata:** Data that helps uniquely identify the object. metadata.name is used to assign a name to the object. metadata.labels is also another really important feature. It not only lets you organise your resources, but it also offers a means to filter your resources, and can be used as a way to refer to a group of resources.

**spec:** The content of this depends on the kind of object in question. Api specifies what structure+content this section should hold.

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


## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)