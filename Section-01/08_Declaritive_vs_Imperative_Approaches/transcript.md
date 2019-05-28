# Imperative vs Declaritive approaches

Structure: slides -> vscode

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
kubectl get pods
kubectl delete pods --all
```

Similar to create a service imperatively we do something like:

```bash
kubectl expose pod pod-httpd --name=svc-nodeport-httpd --type=NodePort --target-port=80 --port=3050 --selector="app=apache_webserver"
kubectl get service
```

Creating objects imperatively without using yaml files does has a few drawbacks:

- The imperative are long and tedious to write. 
- The imperative commands can't be tracked. E.g. if you delete a pod by mistake, then it would a pain to figure out what command you wrote to create that pod in the first place. Whereas yaml files can be version controlled and easy to reapply. 
- Imperative commands are quite restrictive and aren't as versatile. For example in the expose commands I wasn't able to find a node-port port number flag. 


That's why it's actually recommended to create objects using the declarative approach, that is, feeding yaml files into the kubectl apply command, like we did in the hello world demo. 





There is also a 3rd approach to kubernetes object management. This is a hybrid aproach where you feed yaml files into into one of the imperative commands. 


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

In this respect the kubectl apply command is more intelligent, because it is designed to bring a request to a desired state, or do nothing if it is already at that stage. Whereas the verb-based commands only attempts to perform the task its meant to perform. 



So far it might sound like I'm telling you to always, always, always, take the declarative approach. However that's not true, imperative commands and hybrid commands do have it's uses. For example, you can't delete objects using kubectl-apply. Instead you have to use kubectl delete:


```bash
$ kubectl delete pod pod-httpd
pod "pod-httpd" deleted
```

or the hybrid command:


```bash
$ kubectl delete -f configs/
```

