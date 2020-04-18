When a new version of an app comes out, then this new version usually also gets packaged into a new docker image and then uploaded to a docker registry, such as dockerhub. This image usually have the same name, but with a newer tag version.

Whenever that happens, it's common practice to update your replica pods to use the newer image.

Now you might be thinking that you do that by simply updating the yaml file with the new version and then reapply it:

```
# vim
kubectl apply -f replicaset-httpd.yaml
```



However that's not the case. The replicaset's definition does end up changing, but that change doesn't get propagated to the existing replica pods.



In fact, only new pods that the replicaset creates from this point forward will end up being built using the new image.


Now you might think that this is some kind of a bug, but in actual fact, replicasets have been designed to behave that way on purpose. That's because you're supposed to roll out changes to your replica pods using something called kubernetest deployments. Will cover deployments later in the course.



But for now if you did want to get your replicasets to roll out the new image, then you can do that by deleting your pods so that your replicaset builds new replacement pods.

```
kubectl delete pods xxxx
```

You can do this faster by scaling your replicaset to zero and then scale it back up again.

```
kubectl scale replicaset --replicas=0 rs-httpd
kubectl scale replicaset --replicas=5 rs-httpd
```

It's not good practice to do it this way espeically in a production environment since it will cause a downtime.

So as I mentioned earlier, the proper way of rolling out this kind of change is by using Kubernetes Deployments, and we'll cover that later in the course.