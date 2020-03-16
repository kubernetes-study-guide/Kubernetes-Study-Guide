The next thing I want to show you is what happens when I try deleting one of these replica pods.

```
kubectl delete pod xxxx
```

As you can see , as soon as I deleted a pod the replicaset built a new replica pod in it's place.

That's because our replicaset has noticed that we've moved away from the desired state and then took action to fix it. In otherwords, replicasets have builtin self-healing capabilities by design.


This self-healing feature makes replicasets handy even if I only want to create a single pod. Becuase if my single pod dies, then I can rest assured that the replicaset will replace it with a new pod.