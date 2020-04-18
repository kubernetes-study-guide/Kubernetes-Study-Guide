Ok here we have our replicaset along with the pods it created.


Now let's see what happens when I try deleting one of these replica pods.

```
kubectl delete pod xxxx
```

As you can see, as soon as I deleted a pod the replicaset spun up a new replica pod to replace it.

That's because our replicaset has noticed that we've moved away from the desired state and the took the necessary action to fix it.
In otherwords replicaset comes with self-healing capabilities by design.

That's why I tend to use replicasets quite often, even if I only want a single pod, i.e. a replica of one. So that if my single pod dies, then I can rest assured that the replicaset will bring up a new replacement pod.