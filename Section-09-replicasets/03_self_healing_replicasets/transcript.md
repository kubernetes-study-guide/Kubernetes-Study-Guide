Ok here we have our replicaset along with it's 5 replica pods. By the way I've used the sort-by flag to list my pods based on age, starting with the oldest first. And I've also applied the selector filter flag so that it only shows pods that have the label of app, with the value of webserver.


Now let's see what happens when I try deleting a couple of these replica pods.

```
kubectl delete pod xxxx
```

As you can see, as soon as I delete a pod, the replicaset spins up a new replica pod to replace it.

That's because replicaset are constantly monitoring their pods and as soon as we've moved away from the desired state, it straight away took the necessary action to restore the desired state again.
In otherwords replicaset comes with builtin self-healing capabilities by design.

That's why I tend to use replicasets quite often, even if I only want a single pod, i.e. a replica of one. So that if my single pod dies, then I can rest assured that the replicaset will bring up a new replacement pod.