# Stateful Sets

Earlier we came across deployments which are used to manage stateless pods, i.e. pods that don't need any persistant data. 
We also saw that deployments can also manage stateful pods, by taking advantage of Persistant Volumes for externally storing a pod's persistant data. However what if that persistant data relies on the pod's hostname (which in turn is the pod's name) to be static as well as the container's ip address (which in turn is the pods ip addresss) to be static too? Deployment provisioned (and rebuilt) pods have deploymentname-randomstring naming convention. They also gets randomly assigned ip address during rebuilds.



That's where Statefull Sets comes into the picture. StatefullSets are like deployments, but with a few important differences:

- pods built with deployments follows a podname-randomstring convention. But stateful set managed pods have names like podname-0, podname-1, podname-3,...etc naming convention. Therefore their containers
- stateful sets pods comes with builtin persistant storage. The volumes exists even after scaling down
- stateful sets pods always stays on the same worker node that originally hosted them, even if you rebuild them.
- Each stateful set pods comes included with static dns entries, which are of the format:
  podname-number.statefulsetname.namespace.svc.cluster.local 
- pods are created in order. podname-0, podname-1, podname-2....etc. and when scaling down, it does it in reverse order. 


Let's demo this feature by creating the following sts:


```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-httpd
  labels:
    app: httpd_webserver
spec:
  serviceName: httpd-service     # Need to add this in stateful sets
  replicas: 2 
  selector:
    matchLabels:
      app: httpd_webserver
  volumeClaimTemplates:          # this is something specific to StatefulSets
    - metadata:
        name: httpd-data-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:                 
            storage: 10Mi
  template:
    metadata:
      labels:
        app: httpd_webserver
    spec:
      containers:
        - name: cntr-httpd
          image: httpd:2.4.37
          lifecycle:
            postStart:
              exec:
                command:
                  - /bin/bash
                  - -c
                  - |
                    if [ -f /usr/local/apache2/htdocs/index.html ]; then
                      exit 0
                    fi
                    echo "This persistant storage was created on: $(date)" > /usr/local/apache2/htdocs/index.html
                    echo "You've hit container, $(hostname)" >> /usr/local/apache2/htdocs/index.html
          volumeMounts:
            - name: httpd-data-storage
              mountPath: /usr/local/apache2/htdocs/
          ports:
            - containerPort: 80

```

When we apply this we end up with:

```bash
$ kubectl get sts
NAME        READY   AGE
sts-httpd   2/2     7s

$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
sts-httpd-0   1/1     Running   0          30s
sts-httpd-1   1/1     Running   0          28s
```

Notice the sequence based pod naming convention, i.e. no random strings. Stateful Sets are controller objects, but unlike deployments, they manage replica numbers directly, rather than make use of something like replica sets. Let's do a curl test:

```bash
$ curl $(minikube service svc-nodeport-httpd --url)
This persistant storage was created on: Sun Mar 17 23:31:24 UTC 2019
You've hit container, sts-httpd-1
$ curl $(minikube service svc-nodeport-httpd --url)
This persistant storage was created on: Sun Mar 17 23:31:22 UTC 2019
You've hit container, sts-httpd-0
```

Now let's delete a pod manually and see if it get's spun up with the same podname:

```bash
$ kubectl delete pod sts-httpd-0
pod "sts-httpd-0" deleted

$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
sts-httpd-0   1/1     Running   0          4s
sts-httpd-1   1/1     Running   0          15m

```


You'll also see that the timestamp is the same:


```bash
$ curl $(minikube service svc-nodeport-httpd --url)
This persistant storage was created on: Sun Mar 17 23:31:22 UTC 2019
You've hit container, sts-httpd-0
```

That's because the volume persisted. Basically behind the scenes, the volumeClaimTemplates, ended up creating one PVC per sts pod:

```bash
$ kubectl get pvc
NAME                             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
httpd-data-storage-sts-httpd-0   Bound    pvc-c583a5d8-490c-11e9-8c2d-0800274b67c2   10Mi       RWO            standard       21m
httpd-data-storage-sts-httpd-1   Bound    pvc-c6e52bd8-490c-11e9-8c2d-0800274b67c2   10Mi       RWO            standard       21m
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                    STORAGECLASS   REASON   AGE
pvc-c583a5d8-490c-11e9-8c2d-0800274b67c2   10Mi       RWO            Delete           Bound    default/httpd-data-storage-sts-httpd-0   standard                21m
pvc-c6e52bd8-490c-11e9-8c2d-0800274b67c2   10Mi       RWO            Delete           Bound    default/httpd-data-storage-sts-httpd-1   standard                21m
```

These pvc and pv persist even after deleting the pods, and even if you delete everything:

```bash
$ kubectl delete -f configs
```

the only way to delete the pvc+pv, is to do it manually:


```bash
$ kubectl delete pvc httpd-data-storage-sts-httpd-0
persistentvolumeclaim "httpd-data-storage-sts-httpd-0" deleted
$ kubectl delete pvc httpd-data-storage-sts-httpd-1
persistentvolumeclaim "httpd-data-storage-sts-httpd-1" deleted
$ kubectl get pvc
No resources found.
$ kubectl get pv
No resources found.
```


