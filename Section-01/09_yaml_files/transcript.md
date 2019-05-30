# transcript

Structure:
slide
-> website (https://yaml.org/spec/1.2/spec.html) 
-> https://learnxinyminutes.com/docs/yaml/

## slide 

Hello everyone, and welcome back. 


So far we created objects by feeding yaml files into kubectl. But we haven't really explained what yaml files are yet. Yaml is just a standard for writing data in a structured way. It's purpose is similar to other standards such xml and json. YAML is best known for being human readable. 

At it's heart, YAML is based on a key-value pair concept. Where the key is a string, but the value can be thought of as a container. And these containers can hold something simple like a single string, or more key-value pairs. note that yaml makes use of white-space indents as part of it's syntax to distinguish child objects. Keys can also hold a list of values by using this bullet list style dash syntax. 

Therefore a value can essentially hold another yaml code block. And the process can repeat itself further where needed, which means that you end up with a tree like structure. 

Because of this, yaml is ideal for storing complex data in a hierachial structures.  

In the study guide I have included I've links to several websites where you can learn more about the yaml syntax, so that you can quickly get yourself up to speed with it. It's not particarly but it does take a little time to get used to it. 


# slide - list of other names
In Kubernetes, The yaml files we have demoed, are referred to quite generically, as configuration files. However people often refer to these files by other names, such as:

- kubernetes manifests
- or kubernetes yaml descriptors


## vscode

Let's now go over these yaml files in more details. 

By the way, For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

These Kubernetes yaml files have the following general top level keys:


```yaml - have the following open in a text editor
apiVersion: xxx
kind: xxxxx
metadata:
  ...
spec:
  ...
```

apiVersion, kind, metadata, and spec. These keys, with the exception of spec, must be explicitly declared. However the spec section can also be mandatory, depending on the kind of object you're defining. For example, the spec section is mandatory when defining a pod obect, but it's optional when defining namespace objects. We'll cover namespaces later.  

Remember that in kubernetes, the keys are case sensitive. That means you can't write apiVersion all in lower case. 


Now let's go over these 4 keys. 


The 'apiVersion' sets which part of the kubernetes api to access. This depends on the object kind. For example, if the kind is 'Pod' then this field needs to be set to 'v1', You can find out what to set here by using the kubectl-explain command.

```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1
...
```

the 'kind' setting is used to set the kind of object you want to define. So far we have only created pods and services objects, but there's a lot more. You can get a full list of them using the api-resources command:

```bash
kubectl api-resources | less
```

By the way, notice the shortname column. You can use these aliases to cut down on typing. For example to you can run the get services command using the svc alias:


```bash
$ kubectl get svc -o wide
```

The metadata section mainly gets used to store info to help uniquely identify the object, such as the object's name.

The 'spec' section is where you specify the object's detailed specifications. This section varies greatly depending on the type of object your defining.


Ok so now we know what these yaml files are, as well as their highlevel structure. But how do you go about writing them?

One option is that you can create them from scratch, starting with an empty yaml file. In this scenario you would need to use kubectl explain to work out what to write. Theres also the recursive flag that gives a handy high level overview of the various settings as well as the type of value they expect:

```bash
kubectl explain pod --recursive | less
```

You can use the dot notation to drill-down to a particular low level section, to view more detailed info:

```bash
$ kubectl explain pod.spec.containers.image
```

However I'll be honest with you, I hardly ever write these yaml files from scratch. Simply because it's quite time consuming and tedious. Also it's prone to typos, although you can mitigate that by installing varius extensions to your text editor, such as a yaml syntax checker. 


## website 
In practice I just copy and paste sample yaml extracts from the official documentation, and then customise them to meet my needs. There are plenty of examples available.

In case you're planning on taking the CKA exam, you should know that you're not allowed to copy-and-paste more than a couple of lines in the CKA exam. 


## bash
However, in my opinion, the coolest way to create these yaml files is to get kubectl to generate them for you! That's possible by running imperative commands along with the dry-run flag and set the output to yaml. 

```bash
kubectl run pod-httpd --image=httpd --restart=Never --dry-run -o yaml > pod.yaml
```

You can then use these yaml files as boilerplate templates that you can customize to meet your needs. Ok let's do another example but this for a nodeport service:

```bash
kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 --dry-run -o yaml > service.yaml
```

Also, in case you're planning on taking the CKA exam, then this is probably the fastest way to create yaml files during the exam. So it's definitely worth memorising some these commands. The official documentation even has a cheat sheet with several examples of these commands. Just search for cheatsheet using the kubernetes website's search field. 

One final thing I wanted to mention is that so far we created one yaml file per object, however you can define multiple objects in a single yaml file. That's done by using the triple-dash notation as a delimiter. 

```bash
code configs/pod-and-service.yml 
```

Here we have our earlier hello world pod and service definitions, but this time in a single file, with a the triple-dash delimiter seperating them.


```bash
kubectl apply -f configs/pod-and-service.yml 
```


That's it for this video, see you in the next one. 





