# Imperative vs Declaritive approaches

Structure: slides -> vscode -> slides > vscode

## Slides

Hello everyone, and welcome back. 


There's basically two different approaches to creating objects in Kuberenetes. They are the 'Declaritive' and 'Imperative' approaches.


The Declaritive approach is all about using the `kubectl apply` to create your objects. This in turn means that you have to use yaml files to create your objects. 

The imperative approach on the otherhand lets you create, update, and delete your objects without needing to use any yaml files. The imperative approach involves using any of the verb based commands, such as:

- `kubectl run`
- `kubectl create`
- `kubectl expose`
- ...and etc


In our recent hello world demo, we created our pod and nodeport service declaratively. However here's the equivalent imperative command for creating these objects.

## Vscode

For this demo I've opened up a bash terminal inside this video's topic folder. 

```bash
pwd
```

So to create our hello-world pod imperatively, we can use the run command:

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
kubectl get service -o wide
```

As you can see, compared to using kubectl apply, imperative commands can get quite long wieldy. That leads me onto some of the drawbacks of the imperative approach:

## Slides

- The imperative commands are long and tedious to write. 
- The imperative commands can't be tracked. E.g. if you delete a pod by mistake, then it would a pain to figure out what command you wrote to recreate it. Whereas yaml files can be version controlled and easy to reapply. 
- Imperative commands are doesn't have all the available settings accessible. For example in the expose command I demoed just now, I wasn't able to find a node-port port number flag, and there's additional work involved in fixing that problem by using other imperitve commands such as kubectl-edit.


That's why the declarative approach, is the recommended approach, when creating objects in production. I tend to use imperative commands mainly for experimenting with things. 


## Slide - change page

There is actually a 3rd approach to kubernetes object management. This is a hybrid aproach where you use yaml files along with imperative commands. That's done by using the -f flag to feed yamls to your imerative command. 

## VS code
For example here's the hybrid approach to create objects:


```bash
$ kubectl create -f configs/
pod/pod-httpd created
```

By the way, these yaml files are just copies of the yaml files that we used in the earlier hello-world videos. 

However if you run the command again it errors out as indicated by the exit code of 1:

```bash
$ kubectl create -f configs/
Error from server (AlreadyExists): error when creating "configs/pod-httpd.yml": pods "pod-httpd" already exists
$ echo $?
1
```

That's becuase verb based commands aren't as smart as the apply command, that's because verb-based commands just performs the action without first deciding whether it needs to make changes based on the current state. Whereas declarative approach is designed to focus on bring the existing state to the desired state, if that's not already the case. 

So far it might sound like I'm telling you to always, always, always, take the declarative approach. However that's not true, imperative commands and hybrid commands do have it's uses. For example, in the documentation it's recommended to delete objects kubectl delete: 

```bash
$ kubectl delete -f configs/
```


That's it for this video. See you in the next one. 