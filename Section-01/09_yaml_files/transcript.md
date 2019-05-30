# transcript

Structure:
slide
-> website (https://yaml.org/spec/1.2/spec.html) 
-> https://learnxinyminutes.com/docs/yaml/

## slide 

Hello everyone, and welcome back. 


So far we created objects by feeding yaml files into kubectl. But we haven't really explained what yaml files are yet. Yaml is a markup language that is often used as an alternative to things like xml or json. It's used for writing data in a human readable format.

Basically YAML is based on a key-value pair concept. Where the key is a string, but the value can be thought of as a container. And these containers can hold something simple like a single string,  or more key-value pairs, or an list of more values. Basically a value can hold another yaml code block. And the process repeats itself as you drill down to child key-value pairs.

Because of this, yaml is ideal for storing complex data in a hierachial, tree-like structures.  

I've mentioned some websites in the study guide where you can learn more about yaml syntax, and quickly get yourself up to speed with it. 


# slide - list of other names
In Kubernetes, The yaml files we used to create our hello world pod and service earlier, are referred to quite generically, as configuration files. However people often refer to these files by other names, such as:

- kubernetes manifests
- or kubernetes yaml descriptors

In my case I prefer to just call them as just yaml files. 



## vscode
These Kubernetes yaml files have the following top level keys:


```yaml - have the following open in a text editor
apiVersion: xxx
kind: xxxxx
metadata:
  ...
spec:
  ...
```

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

All kubernetes yaml files needs to have 4 high level sections, apiVersion, kind, and metadata, and spec. However the spec section can be optional, depending on the kind of object you want to create. For example, the spec section is mandatory for a pod yaml file, but is optional for a namespace yaml file. We'll cover namespaces later.  

Remember that in kubernetes, the keys are case sensitive. That means you can't write apiVersion all in lower case. 

Also all the indents have to be correct. That's actually a broader yaml requirement rather than a kubernetes specific thing, but I thought I should mention it because it's so subtle, and people often get caught out by that, including myself.


Now let's go over these 4 keys. 


The 'apiVersion' sets which part of the kubernetes api to access. This depends on the object kind. For example, if the kind is 'Pod' then this field needs to be set to 'v1', You can find out what to set here by using the kubectl-explain command.

## swipe to bash
```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1
...
```

## Swipe to vscode
the 'kind' setting is used to set the object type. So far we have only couple of them, pods and services, but there's a lot more. You can run the api-resources command to get a full list of them:

## swipe to bash
```bash
kubectl api-resources | less
```

By the way, notice the shortname column. You can use these aliases to cut down on typing. for example to you can run get service command using the service's alias:


```bash
$ kubectl get svc -o wide
```







## swipe to vscode
the metadata section mainly gets used to store info to help uniquely identify the object, such as the object's name.

The 'spec' section is where you set the object's detailed specifications. Then content under this section varies greatly depending on the type of object your creating. 

You can write multiple object definitions in a single yaml file. 



Ok so now we know what these yaml files are, as well as their highlevel structure. But how do you go about writing them?

First approach is writing them by hand. So you create empty yaml file and then just start writing. In this scenario you would need to use kubectl explain to work out what to write:

```bash
kubectl explain pod | less
```

Theres also the recursive flag that gives a handy high level overview of the various settings as well as the type of value they expect:


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
However, in my opinion, the coolest way to create these yaml files is to get kubectl to generate them for you! That's possible by running imperative commands that, along with the dry-run flag enabled and output flag set to yaml:

```bash
kubectl run pod-httpd --image=httpd --restart=Never --dry-run -o yaml > pod.yaml

kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 > service.yaml
```

With this approach you get to create boilerplate yaml files which you can then customize to meet your needs. 

Also, in case you're planning on taking the CKA exam, then this is probably the fastest way to create yaml files during the exam. So it's definitely worth memorising some these commands. The official documentation even has a cheat sheet with a lot of imperative commands that you can use to create these yaml files.  

One final think I wanted to mention is that so far we created one yaml file per object however you can defined multiple objects in a single yaml file, in which you just need to use triple-dash as a delimiter. 

```bash
code configs/pod-and-service.yml 
```

Here we have our earlier hello world pod and service in a single file, with the triple-dash delimiter to tell kubernetes when one definition ends and another one starts. 


```bash
kubectl apply -f configs/pod-and-service.yml 
```


That's it for this video, see you in the next one. 





