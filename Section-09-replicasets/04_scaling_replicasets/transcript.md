Now there can be times where you want to change the number of replicas pods in your replicaset. To do that you can just edit the replica setting in your replicaset's yaml file, and then reapply it. Or you can use the `kubectl scale` command which is purpose built for this kind of task.

```
kubectl scale --help
```

With this command you have to specify what type of object you want to scale, which in my case is a replicaset, the name of the replicaset, and how many replicas you want, which in my example I'm going to scale down to just a single pod.

```
kubectl scale replicaset rs-httpd --replicas=1
```

This ends up deleting most of the pods, so that only one remains. Ok let's scale back up to 5 pods again:

```
kubectl scale replicaset rs-httpd --replicas=5
```

