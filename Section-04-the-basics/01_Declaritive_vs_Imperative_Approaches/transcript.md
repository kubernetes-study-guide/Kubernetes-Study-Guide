# Imperative vs Declaritive approaches

Structure: slides -> vscode -> slides > vscode

## Slides

Hello everyone, and welcome back. 


There's basically two different approaches to create objects in Kuberenetes. They are the 'Declaritive' and 'Imperative' approaches.


The Declaritive approach is all about using the `kubectl apply` command to create your objects. This in turn means that you have to use yaml files as part of the creaton process. 

The imperative approach on the otherhand lets you create, update, and delete your objects without needing to use any yaml files. This approach involves using one of the verb based commands, such as:

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
kubectl get pods -o wide --show-labels
```

Notice I used the show-labels flags. Thats just to display the labels column in the output.

Similarly, to create a service object imperatively we can use the expose command:

```bash
kubectl expose pod pod-httpd --name=svc-nodeport-httpd --type=NodePort --target-port=80 --port=3050 --selector="app=apache_webserver"
kubectl get service -o wide
```

Now to delete this pod and service, we use the delete command, which is actually another imperative command:

```bash
$ kubectl delete all --all
pod "pod-httpd" deleted
service "kubernetes" deleted
service "svc-nodeport-httpd" deleted
```

Rather than deleting these objects one at a time, I deleted all of them using the 'all' keyword and flag. However by doing that I also inadvertantly ended up deleting the internal kubernetes service. Luckily though, kubernetes will simply recreate that internal service again. 



If you take a look at these imperative commands, you'll notice that imperative commands can get quite long wieldy. That leads me onto some of the drawbacks of the imperative approach:

## Slides

- The imperative commands are long and tedious to write. 
- The imperative commands can't be tracked. For example, if you delete a pod by mistake, then you need to figure out what was the original command you ran to create that pod in the first place and then rerun it again.  Whereas if created the objects using yaml files, then it's just a case of tracking down those files in your git repo, and then run kubectly apply against them.
- Imperative commands doesn't have all the settings available. For example in the expose command I just demoed, I wasn't able to find a node-port port number flag. That means that there's additional work involved in fixing that problem by using other imperitve commands such as kubectl-edit.


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

That's becuase verb based commands aren't as smart as the apply command. They just perform the requested action without first deciding whether it needs doing in the first place. Whereas declarative approach is designed to focus on bring the existing state to the desired state, if that's not already the case. 

So far it might sound like I'm telling you to always, always, always, take the declarative approach. However that's not true, imperative commands and hybrid commands do have it's uses. For example, in the documentation it's recommended to delete objects kubectl delete: 

```bash
$ kubectl delete -f configs/
```

### TODO - Start

One final thing I wanted to show you. You can run a single imperative command to quickly spin up a pod and start a bash session inside it:

```bash
kubectl run podcentos --rm --image=centos --restart=Never -it --command -- /bin/bash
```

Here we've created a pod using the official centos image. This is a handy way to take a look inside an image, without having to go through the hassle of writing out a yaml file first. 

Note that we didn't need to specify the --comand setting here. That's because the dockerfile that built this image already specified bash for the CMD setting. But we've added it in here just so you know what to do if that wasn't the case for the image you want to look inside. 

--rm flag is used to delete the pod after we exit out of the centos bash session. Otherwise if will continue to exist in a failed state. 
###TODO - END


That's it for this video. See you in the next one. 