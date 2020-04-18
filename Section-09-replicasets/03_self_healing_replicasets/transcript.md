Ok here we have our replicaset along with it's 5 replica pods. By the way I've used a number of flags here. The show-labels flag juse displays the labels column, I'm usingthe sort-by flag to list my pods based on age, starting with the oldest first. And I've used the selector flag.  This acts as filter and here I'm telling it to only shows pods that have the label of app, and this applabel has it's value set as webserver.


Now let's see what happens when I try deleting a couple of these replica pods.

```
kubectl delete pod xxxx
```

As you can see, as soon as I delete a pod, the replicaset spins up a new replica pod to replace it.

That's because replicaset are constantly monitoring their pods and as soon as we've moved away from the desired state, it straight away took the necessary action to restore the desired state again.
In otherwords replicaset comes with builtin self-healing capabilities by design.

That's why I tend to use replicasets quite often, even if I only want a single pod, i.e. a replica of one. So that if my single pod dies, then I can rest assured that the replicaset will bring up a new replacement pod.