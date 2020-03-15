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