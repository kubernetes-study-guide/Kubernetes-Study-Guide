Hello everyone and welcome back.

Ok we've covered a lot of really good stuff in the past few videos. However things are going get even more interesting because we're now going to level up and start playing with controller objects.

But what are controller objects I hear you say?


Controller objects are Kubernetes       objects that in turn creates other kubernetes objects. There are a number of controller objects. They include replicasets, deployments, and statefulsets. We'll cover all of them in due course, but for now we'll take a look at replicasets.

A replicaset is used for creating multiple replicas of a pod, hence the name replicaset. You need to create replica pods if you want to implement high-availability and loadbalancing capabilities for your application. We'll cover those capabilities in the video about clusterIP services, later in the course.

Anyway for now here's the replicaset we'll create for this demo:

```
ll
vim replicaset-httpd.yaml
```

Here, we're saying, create an object of the type replicaset, this replicaset should ensure that there are exactly 5 running pods in the set.

This replicaset will have control over any pods with this label. In this exmaple. this label has a key with the name 'app', and it's corresponding value is set to webserver.

Next we have the template section. If you look closely you'll notice that The template section is actually a pod-spec definition nested under it. We're basically telling the replicaset to use this pod spec definition as a template for spinning up the replica pods.

Replicasets makes use of a   label and selectors mechanism to help it keep track of which replica pods it's managing. So for this to work, we have to ensure that our replicaset creates pods that have labels that matches the selector.


Ok that's pretty much a quick overiew of this yaml file.

Now before I apply this yaml file, let's first get a list of our replicasets and pods    :


```
# top right terminal

watch -n kubectl get replicaset -o wide

#bottom right terminal
$ watch -n1 kubectl get pods -o wide
```

As you can see, we don't have any replicaset or pods at the moment.


So let's now go ahead and create our replicaset:

```bash
# bottom left terminal
$ kubectl apply -f my-replicaset.yml
```

Ok I'm going to briefly pause this video for a moment so that you can see what's happening here.



Here we can see that our replicaset desires five running pods since that's what we asked for  in our yaml file. It shows how many pods currently exists and out of those, how many pods have finished initializing and are now running. This matches up with what we see in the other terminal.

Replicasets continuously compares the number of ready pods, to the desired value, and it will create or delete replica pods to reach that desired state. So it means that these three values will eventually line up once the desired state is reached.

Let's now resume the video to witness that in action.


Also notice that the replicaset assigned unique names to the pods by naming them after itself followed by a uniqe string, that's so that each pod has a unique name.

Ok It looks like all the pods are now running, and as expected, the Desired, Current, and Ready Values are now all the same.


Let's confirm these pods are working by curling one of them:

```
# bottom left terminal
$ kubectl run --rm=true -it client --image=centos --restart=Never -- curl http://xxx.xxx.xxx.xxx
```



So far so good. You can also describe your replicaset to get more info:

```
kubectl describe rs xxxx
```


Ok so we've now created our replicaset and confirmed our replica pods are working.

The next thing I want to show you is what happens when I try deleting one of these replica pods.

```
kubectl delete pod xxxx
```

As you can see , as soon as I deleted a pod the replicaset built a new replica pod in it's place.

That's because our replicaset has noticed that we've moved away from the desired state and then took action to fix it. In otherwords, replicasets have builtin self-healing capabilities by design.


This self-healing feature makes replicasets handy even if I only want to create a single pod. Becuase if my single pod dies, then I can rest assured that the replicaset will replace it with a new pod.

Speaking of which, If I want to scale down my current replicaset to a single pod then I need to change the desired number to 1. I can do by updating the yaml file with a new replcas value and then reapply it. However I prefer to use the `kubectl scale` command:

```
kubectl scale replicaset --replicas=1 rs-httpd
```

Ok let's go back to 5 pods again:

```
kubectl scale replicaset --replicas=5 rs-httpd
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



However that's not the case. The replicaset's definition   does end up changin, but that change doesn't get propagated to the existing replica pods. Only new pods that the replicaset creates from this point forward will get created using the update image. One workaround for this is that we can manually trigger pod builds by deleting the existing pods.

```
kubectl delete pods xxxx
```

This approach is repetive and can take a long time if you have a lot of replica pods.

Another option, is to scale down our replicaset to zero and then scale it back up again.

```
kubectl scale replicaset --replicas=0 rs-httpd
kubectl scale replicaset --replicas=5 rs-httpd
```

This is quick and dirty, and it will cause a downtime. So I wouldn't recommend doing this.

The correct way of doing this is by using deployments rather than replicasets.

Deployments is a whole new topic, so we'll take a break here and we'll cover deployments in the next video.
