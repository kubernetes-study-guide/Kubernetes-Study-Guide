# transcript

Structure:
slide
-> vscode
-> github: 

## slide 

Hello everyone, and welcome back. 


So far we've created objects by feeding yaml files into kubectl. But we haven't yet properly explained what yaml files are. Yaml is just a standard for writing data in a structured way. It's purpose is similar to other standards such xml and json. YAML has become really popular thanks it's human readable syntax. 

At it's heart, YAML is based on using key-value pairs. Where the key is a string, but the value can be all kinds of things. For example it can be a simple string, or more key-value pairs. Yaml makes use of white-space in the form of indents as part of it's syntax to show nested child objects. Keys can also hold a list of values by using this bullet list style dash syntax. 

Therefore a value can essentially hold another yaml code block. And the process can repeat itself further where needed, which means that you end up with a tree like structure. 

Because of this, yaml is ideal for storing complex data in a hierachial structures.  

In the study guide I have included I've links to several websites where you can learn more about the yaml syntax, so that you can quickly get yourself up to speed with it. It's not particarly but it does take a little time to get used to it. 


# slide - list of other names
In Kubernetes, The yaml files we have demoed, are referred to quite generically, as configuration files. However people often refer to these files by other names, such as:

- kubernetes manifests
- or kubernetes yaml descriptors
- However I just call them name yaml files. 


## vscode

Let's now go over these yaml files in more details. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

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

apiVersion, kind, metadata, and spec. These keys, with the exception of spec, are mandatory. However the spec section can also be mandatory, depending on the kind of object you're defining. For example, the spec section is mandatory when defining a pod obect, but it's optional when defining namespace objects. We'll cover namespaces later.  

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
kubectl api-resources
```

We have quite a lot of info here, so I'll just scroll up to the start of this output.

As you can see we have a lot of different kinds of objects, and so far we have only touched 2 kinds so far, pods and services. We'll cover more of these kinds as we go through the course. 

By the way, notice the shortname column. You can use these aliases to cut down on typing. For example to you can run the get services command using the svc alias:


```bash
$ kubectl get svc -o wide
```

The metadata section mainly gets used to store info to help uniquely identify the object, such as the object's name.

The 'spec' section is where you specify the object's detailed specs. This section varies greatly depending on the type of object you're defining.


Ok we have now seen what these yaml files are, as well as their high-level structure. But how do you go about writing them?

One option is that you can write them from scratch. In this scenario you start by writing out the 4 high-level keys, then use kubectl-explain to work out what to write under each section. Theres also the recursive flag that gives a handy high level overview of the various settings available:

```bash
kubectl explain pod --recursive
```

We have a huge range of available settings for pods, so I'll just scroll up to the start of this output. 

To learn more about a particular setting, then you can use the dot notation to drill-down, to view more detailed info:

```bash
$ kubectl explain pod.spec.containers.image
```

Writing yaml files by hand like this, is time consuming, but it's a great way to  practice, and you'll learn a lot through trial and error.


Another way to write these manifests, is to just copy and paste sample yaml extracts from the official documentation, and then customise them to meet your needs. There are plenty of examples available.


However my favourite way to write these yaml files, is to get kubectl to generate them for you! That's possible by running imperative commands along with the dry-run flag and yaml output, For example, here's how to create a pod yaml definition: 

```bash
kubectl run pod-name --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
code pod.yaml
```

You can then use these manifests as boilerplate templates as a starting point for  your own yaml files. 

These commands are not that intuitive and are in fact quite tricky to figure out. For example it took me several minutes of googling before realising that I needed to include this restart flag settng. If I leave out this flag then I end up with a different kind of object altogether!

# github
That's why in the Appendix section of the study guide, I have added an article called 'generate yaml using kubectl'. This article has a list of example commands that you can use to generate yaml files for various objects types.

I highly recommend memorising some of these commands if you're planning to take the CKA exam. That's because you won't have time to write yaml files manually, and also you're not allowed copy and paste more than 2 lines at a time during the exam, that rules out copying yaml samples from the official documentation. 

## vscode
One final thing I wanted to mention is that so far we created one yaml file per object, however you can define multiple objects in a single yaml file. That's done by using the triple-dash notation as a delimiter. 

```bash
code configs/pod-and-service.yml 
```


However if you're planning on taking the CKA exam, then you can't take this approach, that's because you're not allowed to copy-and-paste more than a couple of lines at a time in the CKA exam. 

Here we have our earlier hello world pod and service definitions, but this time in a single file, with a the triple-dash delimiter seperating them. This triple-dash notation is part of the standard yaml syntax. 



```bash
kubectl apply -f configs/pod-and-service.yml
kubectl get all
```


That's it for this video, see you in the next one. 





