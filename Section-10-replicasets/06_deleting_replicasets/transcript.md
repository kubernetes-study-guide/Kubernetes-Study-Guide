We're going to carry on with the previous demo.

Now what if you want to delete all your replica pods, you can do that by deleting the replicaset.


```
kubectl delete replicaset xxxx
```

That ends up deleting the replicaset along with all it's pods.


There's also option available, which is that you keep the replicaset but delete all it's pods. Let's create our replicaset again so that I can show how that's done.

```
kaf -f ...
```

Now if you just want to delete all the replica pods but keep the replicaset, then you can do that by simplying scaling your replicaset down to zero pods.


```
kubectl scale replicaset --replicas=0 rs-httpd
```