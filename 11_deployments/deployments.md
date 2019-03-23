# Deployments

[deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) are a special type of object referred to as 'controllers'. Controllers are objects that monitors+controls other objects.

In the case of deployments, they control the state of other pod objects. Deployments are a bit like the equivalent of AWS EC2 Autoscaling Scaling Groups, Where instead of autoamatically scaling ec2 instances, in kubernetes you autoscale identical pods across one or more worker nodes.

Deployments monitors the pods by continously performing pod healthchecks and ensures that the desired state (of number of healthy active pods) is maintained. That's why it's [kubernetes best practice](https://kubernetes.io/docs/concepts/configuration/overview/#naked-pods-vs-replicasets-deployments-and-jobs) to always create pods using a controller object, such as deployments.

Deployments are effectively a wrapper for replicasets behind the scenes (with a few extra bells and wistles). Here's our example deployment yaml file:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-httpd
  labels:
    component: httpd_webserver
spec:
  replicas: 2
  minReadySeconds: 60 # This is one of the bells and whistle feature that deployment objects provides,
                      # but replica sets doesn't.
                      # it's how many seconds to wait after pod is created, before deployment
                      # will allow the pod to start receiving traffic.
  selector:
    matchLabels:
      component: httpd_webserver
  template:
    metadata:
      labels:
        component: httpd_webserver
    spec:
      containers:
        - name: cntr-nginx
          image: httpd:latest
          ports:
            - containerPort: 80
```

applying this yaml file ends up creating the following deployment object:

```bash
$ kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
dep-httpd   2/2     2            2           12s
```

 This deployment object created the following replicaset:

```bash
$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
dep-httpd-6b84f9fd8c   2         2         2       16s
```

This replicaset in turn created the following pods:

```bash
$ kubectl get -o wide pods
NAME                                READY     STATUS    RESTARTS   AGE       IP           NODE
deployment-httpd-648756c8dd-hcc7f   1/1       Running   0          1m        172.17.0.5   minikube
```

So a deployment object created another controller object (replicaset), which in turn created the pod objects. That means if we manually delete the RS then the deployment would automatically recreate it again, which in turn will recreate the pods. Going back to the yaml file, you'll notice that it's content is essentially 3 object definitions in a nested fashion. At the top we have the deploymnet, followed by the replicaset, and finally the pod definition. If you want to increase the number of pods running under this deployment, then just update the replicas setting in the yaml file and reapply. You can check the sequence of tasks that kubernetes performed behind the scenes using the 'events' subcommand:


```bash
$ kubectl get events
LAST SEEN   TYPE     REASON              KIND         MESSAGE
38m         Normal   Scheduled           Pod          Successfully assigned default/dep-httpd-95466f584-k2t4v to minikube
38m         Normal   Pulling             Pod          pulling image "httpd:2.4.37"
38m         Normal   Pulled              Pod          Successfully pulled image "httpd:2.4.37"
38m         Normal   Created             Pod          Created container
38m         Normal   Started             Pod          Started container
38m         Normal   Scheduled           Pod          Successfully assigned default/dep-httpd-95466f584-nvl26 to minikube
38m         Normal   Pulling             Pod          pulling image "httpd:2.4.37"
38m         Normal   Pulled              Pod          Successfully pulled image "httpd:2.4.37"
38m         Normal   Created             Pod          Created container
38m         Normal   Started             Pod          Started container
38m         Normal   SuccessfulCreate    ReplicaSet   Created pod: dep-httpd-95466f584-k2t4v
38m         Normal   SuccessfulCreate    ReplicaSet   Created pod: dep-httpd-95466f584-nvl26
38m         Normal   ScalingReplicaSet   Deployment   Scaled up replica set dep-httpd-95466f584 to 2
```




## Rolling updates to pods

When a new image version is released, then it's likely that you want to update all your deployment-managed pods to have containers built with the new image. With deployments it's quite easy to deploy the new image. You simply update the yaml file, then reapply. The rollout can take a little while, but you can monitor the deployment in realtime like this:

```bash
$ kubectl rollout status deployment dep-httpd
Waiting for deployment "dep-httpd" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "dep-httpd" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "dep-httpd" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "dep-httpd" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "dep-httpd" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "dep-httpd" rollout to finish: 1 old replicas are pending termination...
deployment "dep-httpd" successfully rolled out
```

Basically the deployment will create a brand new replicaset with the new image defined inside it. Then gradually scale it up, while it scales down the existing replicaset to zero.

```bash
$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
dep-httpd-7b9d4f7ccf   2         2         2       6m18s
dep-httpd-95466f584    1         1         1       149m
```

The net result is that all the old pods gets replaced by the new pods one at a time (also you can further customise the rolling update strategy using maxSurge+maxUnavailable yaml settings so that you can replace pods 2 or more pods at a time). 

## Perform rollbacks

The cool thing about this approach is that if you decide to do a rollback, then you simply need to revert the yaml file back to it's old image and reapply it, but this time the deployment won't create a new replicaset, instead it would just scale up the existing replicaset, and scale down the now redundant replicaset. However if want need to rollback in rush and don't have time to update the yaml file, then you can use the 'rollout' to achieve the same effect:

```bash
$ kubectl rollout undo deployment dep-httpd
deployment.extensions/dep-httpd rolled back
```

The rollout command also has number of other subcommands that could come in handy:

```bash
$ kubectl rollout
history  pause    resume   status   undo
```

If you want to roll back several revisions back then first track down which revision you want:

```bash
$ kubectl rollout history deployment dep-httpd
deployment.extensions/dep-httpd 
REVISION  CHANGE-CAUSE
4         <none>
5         <none>
```

Note: by default kubernetes only stores info about the last 2 deployments. which is why revisions 1-3 are missing here. You can increase this limit by settin your deployment's deployment.spec.revisionHistoryLimit yaml setting.

Let's say I want to rollback to the image used in revision 4, then first need to check what image I will end up with:

```bash
$ kubectl rollout history deployment dep-httpd --revision=4
deployment.extensions/dep-httpd with revision #4
Pod Template:
  Labels:       component=httpd_webserver
        pod-template-hash=7b9d4f7ccf
  Containers:
   cntr-httpd:
    Image:      httpd:2.4.38
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

Once I'm happy with my chosen revision I then perform the rollback:

```bash
$ kubectl rollout undo deployment dep-httpd --to-revision=4
deployment.extensions/dep-httpd rolled back
```

## Deleting Deployments

If you want to delete the deployment and everything that's associated with it, then do:

```bash
kubectl delete deployments deployment-nginx
```

Notice that we run this command imperitively rather than doing it declaritevely by specifying the config file.