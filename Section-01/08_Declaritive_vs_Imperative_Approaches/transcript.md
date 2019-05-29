# Imperative vs Declaritive approaches

Structure: slides -> vscode -> slides > vscode

There's basically two different approaches to creating objects in Kuberenetes. They are the 'Declaritive' and 'Imperative' approaches.


The Declaritive approach is all about using the `kubectl apply` to create your objects. This in turn means that you have to use yaml files to create your objects. 

The imperative approach on the otherhand lets you create, update, and delete your objects without needing to use any yaml files. The imperative approach involves using any of the verb based commands, such as:

- `kubectl run`
- `kubectl create`
- `kubectl expose`
- ...and etc


In our recent hello world demo, we created our pod and nodeport service declaratively. However here's the equivalent imperative command for creating our hello world pod:


```bash
kubectl run pod-httpd --image=httpd --labels="app=apache_webserver" --restart=Never
kubectl get pods pod-httpd -o wide --show-labels
```

Now to delete the pod we use the delete command, which is actually another imperative command:

```bash
kubectl delete pods pod-httpd
```

Similarly, to create a service imperatively we can use the expose imperative command:

```bash
kubectl expose pod pod-httpd --name=svc-nodeport-httpd --type=NodePort --target-port=80 --port=3050 --selector="app=apache_webserver"
kubectl get service
```

As you can see, compared to using kubectl apply, imperative commands can get quite long wieldy. That leads me onto some of the drawbacks of the imperative approach:

- The imperative commands are long and tedious to write. 
- The imperative commands can't be tracked. E.g. if you delete a pod by mistake, then it would a pain to figure out what command you wrote to recreate it. Whereas yaml files can be version controlled and easy to reapply. 
- Imperative commands are doesn't have all the available settings accessible. For example in the expose command I demoed just now, I wasn't able to find a node-port port number flag, and there's additional work involved in fixing that problem by using other imperitve commands such as kubectl-edit.


That's why the declarative approach, is the recommended approach, when creating objects in production. I tend to use imperative commands mainly for experimenting with things. 




There is actually a 3rd approach to kubernetes object management. This is a hybrid aproach where you use yaml files along with imperative commands. That's done by the -f flag to feed yamls to your imerative:


```bash
$ kubectl create -f configs/
pod/pod-httpd created
```

However if you run the command again it errors out as indicated by the exit code of 1:

```bash
$ kubectl create -f configs/
Error from server (AlreadyExists): error when creating "configs/pod-httpd.yml": pods "pod-httpd" already exists
$ echo $?
1
```

In this respect the kubectl apply command is more intelligent, because it ensures a desired state is reached by taking the necessary actions, or do nothing if it is already at that state. Whereas the verb-based commands only attempts to perform the action irrespective it needs to or not. 



So far it might sound like I'm telling you to always, always, always, take the declarative approach. However that's not true, imperative commands and hybrid commands do have it's uses. For example, you can't delete objects using kubectl-apply. Instead you have to use kubectl delete:


```bash
$ kubectl delete pod pod-httpd
```

or the hybrid command:


```bash
$ kubectl delete -f configs/
```

However in this course I will stick to the declarative approach when creating files. 