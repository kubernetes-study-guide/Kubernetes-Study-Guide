You can delete a replicaset using the usual kubectl delete command.


```
kubectl delete replicaset xxxx
```

As you have probably guessed, This ends up deleting the replicaset along with all it's replica pods.


However there's a slightly different approach you can take, lets bring back my replicaset so I can show you what I mean. Now let's say I want to delete all the replica pods but keep the replicaset. In that case I can just scale my replicaset to 0 pods:

```
kubectl scale replicaset rs-httpd --replicas=0
```

This is a bit like deactivating a replicaset without actually deleting. This approach can be useful in case you want to go back to using this replicaset in future.