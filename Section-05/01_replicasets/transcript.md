Hello everyone and welcome back.

Wow! You made it to section 3 of the course! That's awesome! If you've been 


We're now going to level up and start taking a look at controller objects. So what are Controller objects I hear you say? 


Well, in Kubernetes we can create special objects that in turn are responsible for creating and managing other kubernetes objects. That's why these special objects are referred to as controller objects. A few examples of controller objects are replicasets, deployments, and statefulsets. We'll cover all of them in due course, but for now we'll take a look at replicasets.

A replicaset is used for creating multiple replicas of a pods, hence the name replicaset. You need to create replica pods if you want to implement failover and loadbalancing capabilities. We'll cover how those capabilities work later on. 

Anyway for now here's the replicaset we'll create for this demo:

```
code configs/replicaset-httpd.yaml
```


Here, we're saying, create an object of the type replicaset, this replicaset should ensure that there are exactly 3 running pods in the set.

This replicaset will have control over any pods with this label. In this exmaple. this label has a key with the name 'app', and it's corresponding value is set to httpd_webserver. 

Next we have the template section. This section is actually pod-spec definition nested under it, so this structure should look familiar to you. We're basically telling the replicaset to use this pod spec definition as a template for spinning up the replica pods. 

By the way I've added a custom command and args, I'll explain whats going on here a bit later. 


Replicasets makes use of the label and selectors mechanism to help it keep track of which replica pods it's managing. So for this to work, we have to ensure that replicaset creates pods that have labels that matches the selector. 

Now before I demo this yaml file, let's first get a list of our pods. 


```
# open another bash terminal in vscode using split vertical icon
$ clear 
$ watch kubectl get pods
```

By the way, I'm using the watch command here. The watch command runs whatever comes after it every couple of seconds. So in this instance its going to show the output of kubectl get pods, and refresh that info every 2 seconds. so let's see what that looks like:

```
press enter
```

As you can see, At the moment we don't have any pods at the moment. So let's now create our replicaset:

```bash
$ kubectl apply -f my-replicaset.yml
```

We can now see our pods are now coming into existance. Let's take a look at the replicaset itself:

```
$ kubectl get replicaset -o wide
NAME       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
rs-httpd   3         3         3       22m   cntr-httpd   httpd:latest   app=httpd_webserver
```

Ok I'm going to briefly pause the video so I go over what's going on here. 


This shows the current status of our replicaset. Our replicaset desires five running pods because that's what we specified in our yaml file. It shows how many pods currently exists and out of those, how many pods have finished initializing and are now running. Replicasets continuously compares the number of ready pods, to the desired value, and it will create or delete replica pods to reach the desired state. So it means that these 3 values will eventually line up once the desired state is reached. We can also describe the replicaset to get even more detail. So lets resume the video to show that. 

```
kubectl describe rs xxxx
```

Here it says that it's created 5 pods. These pods are named after the replicaset followed by a uniqe string, that's so that each pod has a unique name.

It looks like all the pods are now running, so let's go back and check the replicaset's status again:

```
$ kubectl get replicaset -o wide
NAME       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
rs-httpd   3         3         3       22m   cntr-httpd   httpd:latest   app=httpd_webserver
```

We can now see the number of pods that are desired, current, and ready, are all the same, which is perfect. 

Now let's see if our replica pods are working by testing one of them. In our case we can do that by curling the ip address for one of our replica pods:

```
kubectl get pods -o wide
```

For our test we'll run the curl command from inside a testpod:

```
$ kubectl run testpod --rm --image=centos --restart=Never -it --command -- curl http://172.17.0.12
```
# or demo kube-proxy command, or port-forward command. 

Awesome that worked. But not only that, unlike in previous demos where we used to get a generic 'it's working' message. This time it printed out the pod's name. That's thanks to the tweak I'm made to the docker image's startup script. The httpd image by default is preconfigured to just run the httpd-foreground command, which I found by looking up the httpd image's oficial Dockerfile on Github. I still want to run this command which is why it's still shown here, but just before starting it, I've inserted a command to update the  index.html file's content, so that it includes the pod's hostname. Pretty cool right. 


Ok so we've now created our replicaset and confirmed our replica pods are working. 

Now let's see what happens if I try deleting one of the replica pods and see what happens. 

```
kubectl delete pod xxxx
```

As you can see on the left, as soon as we deleted that pod the replicaset built a new replica pod in it's place.

That's because our replicaset has noticed that we've moved away from the desired state and then took action fix it. 


That means when you create your pods using a replicaset, you also end up adding Self-healing capabilities for your pods too. This is really cool, to me it's almost as cool as how Luke Skywalker brings balance to to the force in Star wars.

This self-healing capability makes replicasets handy even if I only want to create a single pod. So that if your single pod dies, then replicaset will replace it with a new pod.  

Speaking of which I want to scale down my current replicaset to a single pod then I need to change the desired number to 1. You can do that by updating the replica value to 1 in your yaml file and then reapply it. However I prefer to do`kubectl scale command`:

```
kubectl scale replicaset
```

Now what if you want to delete all your replica pods, you can do that by deleting the replicaset. However If you want to delete all the pods without deleting your replicaset, then you can do that by simplying scaling your replicaset down to zero pods. 


```
kubectl scale replicaset --replicas=0 rs-httpd
```

There's another type of controller object that's very similar to replicasets, and that's deployments. In fact out of the 2, deployments objects is the preferred way for creating your pod clusters. We'll cover deployments in the next video   


Ok that's it for this video. See you in the next one. 
