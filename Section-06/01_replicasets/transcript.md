Hello everyone and welcome back.

Ok we've covered a lot of really good stuff in the past few sections. However things are going get even more interesting because we're now going to level up and start playing with controller objects.

But what are they I hear you say? 


Controller objects are objects that in turn creates other kubernetes objects. There are a number of objects types that falls into the controller category. They include replicasets, deployments, and statefulsets. We'll cover all of them in due course, but for now we'll take a look at replicasets.

A replicaset is used for creating multiple replicas of a pod, hence the name replicaset. You need to create replica pods if you want to implement high-availability and loadbalancing capabilities for your application. We'll cover those capabilities in the clusterIP service video, later in the course. 

Anyway for now here's the replicaset we'll create for this demo:

```
vim replicaset-httpd.yaml
```

Here, we're saying, create an object of the type replicaset, this replicaset should ensure that there are exactly 5 running pods in the set.

This replicaset will have control over any pods with this label. In this exmaple. this label has a key with the name 'app', and it's corresponding value is set to httpd_webserver. 

Next we have the template section. This section is actually a pod-spec definition nested under it. We're basically telling the replicaset to use this pod spec definition as a template for spinning up the replica pods. 

By the way I've added a custom command and args, I'll explain whats going on here a bit later. 


Replicasets makes use of the label and selectors mechanism to help it keep track of which replica pods it's managing. So for this to work, we have to ensure that our replicaset creates pods that have labels that matches the selector. 


Ok that's pretty much a quick overiew of this yaml file. 

Now before I apply this yaml file, let's first get a list of our pods. I'll do that in a another terminal:


```
# open another bash terminal in vscode using split vertical icon
$ clear 
$ watch --interval 1 'kubectl get pods'
```

Notice I'm using the 'watch' command here. This utility is used for running a command, over and over again. So in this example I've instructed the watch command to run `kubectl get pods` every second and show it's output. Watch wasn't available on my macbook so I had to install it using brew (pop up box: brew install watch):

```
press enter
```


As you can see, we don't have any pods at the moment. The same is also true for replicasets:

```
kubectl get replicaset
```

I'll also open a third terminal to monitor the events log. 

```
kubectl get events --watch
```

Kubectl actually comes with a builtin --watch flag that does a similar job as the watch command I'm using up here. Ok I'll let's hit enter. 


```
<enter>
```

Ok I can see the event logs. 









So let's now go ahead and create our replicaset:

```bash
$ kubectl apply -f my-replicaset.yml ; watch --interval 1 kubectl get replicaset
```

Here I've actually typed 2 commands on a single line, seperated by a semi colon. This results in the second command running straight after the first command. 

I'm doing this because it takes a little while for the replicaset to get ready and I wanted you to see it's progress as it's starting up. Ok let's hit enter. 

Ok I'm going to briefly pause this video for a moment so that you can see what's happening here. 

```
$ kubectl get replicaset -o wide
NAME       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
rs-httpd   3         3         3       22m   cntr-httpd   httpd:latest   app=httpd_webserver
```



On the left terminal it shows that our replicaset desires five running pods since that's what we specified in our yaml file. It shows how many pods currently exists and out of those, how many pods have finished initializing and are now running. This matches up with what we see on the right terminal. 

Replicasets continuously compares the number of ready pods, to the desired value, and it will create or delete replica pods to reach that desired state. So it means that these three values will eventually line up once the desired state is reached. 

Let's now resume the video to witness that in action. 


Also notice that the replicaset assigned unique names to the pods by naming them after itself followed by a uniqe string, that's so that each pod has a unique name.

Ok It looks like all the pods are now running, and as expected, the Desired, Current, and Ready Values are all the same. 

Ok I'm going to leave these two terminals running and open up a third terminal. 



We can also describe the replicaset to get even more detail, for example it shows in what order the replicaset created these pods: 

```
kubectl describe rs xxxx
```



Now let's check if our replica pods are working. In our example we can do that by curling the ip address for one of our replica pods:

```
kubectl get pods -o wide
```



For our test we'll run the curl command from inside a testpod. So let's create our good old dummy pod using this yaml file:

```
cat ....
kubectl apply -f .....
```

Now let's exec into our dummypod and curl the ip address of one of our replica pods. 

```
kubectl exec ...
curl ....
```

# this approach should be covered in a later video:
```
$ kubectl run testpod --rm --image=centos --restart=Never -it --command -- curl http://172.17.0.12
```
this should be covered in an earlier vidoe. 
```
kubectl port-forward pod-name 8080:80
```

Awesome that worked. But not only that, unlike in previous demos where we used to get a generic 'it's working' message. This time it printed out the pod's name. That's thanks to the tweak I'm made to the docker image's startup script. The httpd image is preconfigured to just run the httpd-foreground command. I found that out by looking up the httpd image's official Dockerfile on Github. I've modified this slightly by inserting a command to update the index.html file's content, so that it includes the pod's hostname. Pretty cool right. 


Ok so we've now created our replicaset and confirmed our replica pods are working. 

The next thing I want to show you is what happens when I try deleting one of these replica pods. 

```
kubectl delete pod xxxx
```

As you can see on, as soon as we deleted that pod the replicaset built a new replica pod in it's place. 

That's because our replicaset has noticed that we've moved away from the desired state and then took action to fix it. In otherwords, replicasets have builtin self-healing capabilities by design. 


This is really cool, to me it's almost as cool as how Luke Skywalker brings balance to to the force.

This self-healing capability makes replicasets handy even if I only want to create a single pod. So that if my single pod dies, then the replicaset will replace it with a new pod.  

Speaking of which I want to scale down my current replicaset to a single pod then I need to change the desired number to 1. You can do that by updating the replica value to 1 in your yaml file and then reapply it. However I prefer to use the `kubectl scale` command:

```
kubectl scale replicaset --replicas=1 rs-httpd
```

As you can see, we now have a one replica pod. Ok I'll scale back up to 5 pods again. 

```
kubectl scale replicaset --replicas=1 rs-httpd

```






Now what if you want to delete all your replica pods, you can do that by deleting the replicaset. However If you just want to delete all the replica pods but keep the replicaset, then you can do that by simplying scaling your replicaset down to zero pods. 


```
kubectl scale replicaset --replicas=0 rs-httpd
```

Ok I'll scale back up to 5 pods agains. 

```
kubectl scale replicaset --replicas=5 rs-httpd
```


Now what about if you want make change to your replicaset. For example update the image version. In this scenario you might be thinking that it's just a case of updating the yaml file with the new version and then reapply it, like this:

```
kubectl apply -f replicaset-httpd.yaml
```



However that's not the case. It makes changes to the replicaset fine, but that change doesn't get propagated to the replica pods. The change only comes when the replicaset builds any pod from this point onwards. So we can manually trigger that by deleting the existing pods. 

```
kubectl scale replicaset --replicas=0 rs-httpd
```

As you can see from the events log, our kube cluster is now pulling down the new image from dockerhub in order to build the new replica pod. 


However this approach is clunky and tedious. That's why in practice it's recommended to use deployments instead of replicasets. We'll cover deployments in the next video. 


Ok that's it for this video. See you in the next one. 
