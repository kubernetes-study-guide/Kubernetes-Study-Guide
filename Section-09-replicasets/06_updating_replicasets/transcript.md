When a new version of an app is released, the app's developers also usually package the new app into a new docker image and upload it with a new tag to a image registry, such as docker hub


Whenever that happens, it's common practice for everyone to recreate their pods using the newer image.

Now you might be thinking that you can do that by simply updating the yaml file with the new tag and then reapply it, so let's try that now:

```
# vim
kubectl apply -f replicaset-httpd.yaml
```

In our case I'll change this from 38 to 39. Then reapply it.


The replicaset has now been updated to reflect the new image tag.


However that change hasn't propagated to the replica pods.

```
kubectl get pods xxxx -o yaml | grep image
```

As you can can see here, it's still using the old image.



That's because this change only takes effect for any new pods the replicaset creates from this point forward.


Now you might think that this is some kind of a bug, but in actual fact, replicasets have been designed to behave that way on purpose. That's because you're supposed to roll out changes to your replica pods using something called kubernetes deployments. We'll cover deployments in the next video.



But for now if you did want to force yoru replicaset to roll out the new image, then you can do that by deleting your pods so that your replicaset builds new replacement pods.

```
kubectl delete pods xxxx
```

You can do this faster by scaling your replicaset to zero and then scale it back up again.

```
kubectl scale replicaset --replicas=0 rs-httpd
kubectl scale replicaset --replicas=5 rs-httpd
```

It's not good practice to do it this way especially in a production environment since it will cause a downtime.

The proper way of rolling out this kind of change is by using Kubernetes Deployments, and we'll cover that next.