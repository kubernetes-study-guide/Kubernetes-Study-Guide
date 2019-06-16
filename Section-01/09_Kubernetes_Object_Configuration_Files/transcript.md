# transcript

TODO: Need to break this topic/video up into 3 seperate videos:
- yaml file intro (include particular syntax used in this course.)
- Anatomy of a object configuration file. - E.g. expand on this.  apiVersion gives you access to new beta features. Write the structure as you go along. 
- how to write the the yaml files (demo for kubectl explain) and how to quickly genererate them. 


Structure:
slide
-> vscode
-> github: 

## slide 

Hello everyone, and welcome back. 


So far we've created objects by feeding yaml files into kubectl. But we haven't yet properly explained what yaml files are. Yaml is just a standard for writing data in a structured way. It's similar  to other standards such xml and json. However  YAML has become really popular thanks it's human readable syntax. 

At it's heart, YAML is based on     key-value pairs. Where the key is a string, but the value can be all kinds of things. For example it can be a simple string, or more key-value pairs. Yaml makes use of white-space in the form of indents as part of it's syntax to show nested child objects. Keys can also hold a list of values as indicated by this hyphen syntax, where each value can be yet more key-values 

Therefore a value can essentially hold another yaml code block. And the process can repeat itself further where needed, which means that you end up with a tree like structure. 

Because of this, yaml is ideal for storing complex data in a hierachial structures.  

In the study guide I have included links to several websites where you can learn more about the yaml syntax. These guides will help you get yourself up to speed with writing yaml files. It's not particarly hard,  but it does take a little time to get used to it. 


# slide - list of other names
In Kubernetes, The yaml files we have demoed so far, are referred to quite generically, as object configuration files. However people often refer to these files by other names, such as:

- kubernetes manifests
- or kubernetes yaml descriptors
- or yaml definitions
- or just       yaml files. 


## vscode

Let's now go over these yaml files in more details. 

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
clear
command+K shortcut
kubectl delete all --all
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

apiVersion, kind, metadata, and spec. These keys, with the exception of spec, are mandatory. However the spec section can also be mandatory, depending on the kind of object you're defining. For example, the spec key     is mandatory when defining a pod obect, but it's optional when defining namespace objects. We'll cover namespaces later.  

Remember that in kubernetes, the keys are case sensitive. That means you can't write apiVersion all in lower case. 


Now let's go over these 4 keys. 


The 'apiVersion' sets which part of the kubernetes api to access. This depends on the object kind. For example, if the kind is 'Pod' then this field needs to be set to 'v1', You can find out what to set here by using the "kubectl explain"  command.

```bash
clear
command+K shortcut
$ kubectl explain pod
KIND:     Pod
VERSION:  v1
...

then scroll up using scrollbar
```

Next we have the 'kind ' key. This setting is used to specify the kind of object you want to define. So far we have only created pod and service objects    , but there's a lot more. You can get a full list of them using the api-resources command:

```bash
clear
command+K shortcut
kubectl api-resources

then scroll up using scrollbar
```

We have quite a lot of output here, so I'll just scroll up to the beginning .


As you can see we have a lot of different kinds of objects, and we have only touched on  2 kinds so far, pods and services. We'll cover more of these kinds as we go through the course. 

By the way, notice the shortname column. You can use these aliases to cut down on typing. For example to you can run the get services command using the svc alias:


```bash
$ kubectl get svc -o wide
```

Next we have metadata. The section is mainly used for storing info to help uniquely identify the object, such as the object's name.

The 'spec' section is where you specify the object's detailed specs. This section varies greatly depending on the type of object you're defining.


Ok we have now seen what these yaml files are, as well as their high-level structure. But how do you go about writing them?

One option is that you can write them from scratch. In this scenario you start by creating an empty text file with a dot yaml extension. Then you write out the 3 high-level mandatory keys. After that you can use kubectl-explain to work out what to write next. Theres also the recursive flag that gives a handy high level overview of the various settings available:

```bash
clear
command+K shortcut
kubectl explain pod --recursive
```

We have a huge range of settings for pods, so I'll just scroll up to the beginning. 

To learn more about a particular setting,    you can use the dot notation to drill-down, to view more detailed info:

```bash
kubectl explain pod.spec.containers.image
```

Writing yaml files by hand like this, is time consuming, but it's a great way to practice, and you'll learn a lot through trial and errer.


Another way to write these manifests, is to just copy and paste sample yaml extracts from the official documentation, and then customise them to meet your needs. 


However my favourite way to write these yaml files, is to get kubectl to generate them for you! That's possible by running imperative commands along with the dry-run flag and yaml output, For example, here's how to create a pod yaml definition: 

```bash
kubectl run pod-name --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
code pod.yaml
```

You can then use these generated manifests as a starting point for writing your own yaml files. 

These commands are not that intuitive,   in fact quite tricky to figure out. For example it took me several minutes of googling before realising that I needed to include this restart flag settng. If I leave out this flag then I end up with a different kind of object altogether!

# github
That's why in the Appendix section of the study guide, I have added a    page with a list of commands for generating these various boilerplate yaml files. The page is called 'generate yaml using kubectl'.

I highly recommend memorising some of these commands if you're planning to take the CKA exam. That's because you won't have time to write yaml files manually, and also you're not allowed copy and paste more than 2 lines at a time during the exam. 

## vscode
One final thing I wanted to mention is that so far we created one yaml file per object, however you can define multiple objects in a single yaml file. That's done by using the triple-dash notation as a seperator. 

```bash
code configs/pod-and-service.yml 
```

Here we have the hello world pod and service definitions from our   earlier demos, but this time in a single file, with a the triple-dash seperating them. This triple-dash notation is part of the standard yaml syntax. So when you apply this file you end up with the same results as we saw earlier.   



```bash
kubectl apply -f configs/pod-and-service.yml
kubectl get all
```


That's it for this video, see you in the next one. 





