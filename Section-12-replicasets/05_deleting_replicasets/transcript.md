Now what if you want to delete all your replica pods, you can do that by deleting the replicaset.


```
kubectl delete replicaset xxxx
```

That ends up deleting the replicaset along with all it's pods.


However if you want to keep the replicaset but delete all it's pods then you can do that by simply scaling your replicaset to 0


```
kubectl scale replicaset --replicas=0 rs-httpd
```