Speaking of which, If I want to scale down my current replicaset to a single pod then I need to change the desired number to 1. I can do by updating the yaml file with a new replcas value and then reapply it. However I prefer to use the `kubectl scale` command:

```
kubectl scale replicaset --replicas=1 rs-httpd
```

Ok let's go back to 5 pods again:

```
kubectl scale replicaset --replicas=5 rs-httpd
```