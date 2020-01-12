# Accessing pod metadata using volumes

Env data only gives you access to a limited amount of data, in particular it doesn't give you access to pod **labels** and **annotations** data. If you want to give your pods access to more data, then you do that using the `pod.spec.volumes.downwardAPI` volume.


```yaml
---
apiVersion: v1
kind: Pod 
metadata:
  name: pod-httpd
  annotations:
    annotation1: hello
    annotation2: world
  labels:
    app: httpd
    cms: wordpress
spec:
  containers:
    - name: cntr-httpd
      image: httpd:latest 
      ports:
        - containerPort: 80
      volumeMounts:
        - name: labels-and-annotations
          mountPath: /etc/pod_data
  volumes:
    - name: labels-and-annotations
      downwardAPI:
        items:
          - path: labels_data.txt
            fieldRef:
              fieldPath: metadata.labels
          - path: annotations_data.txt
            fieldRef:
              fieldPath: metadata.annotations
```

After creating the pod, you should now find labels and annotation data available in text files inside the pod:

```bash
$ kubectl exec pod-httpd -c cntr-httpd -it -- bash
root@pod-httpd:/usr/local/apache2# cd /etc/pod_data

root@pod-httpd:/etc/pod_data# ls -l
total 0
lrwxrwxrwx 1 root root 27 Apr  5 10:03 annotations_data.txt -> ..data/annotations_data.txt
lrwxrwxrwx 1 root root 21 Apr  5 10:03 labels_data.txt -> ..data/labels_data.txt

root@pod-httpd:/etc/pod_data# cat annotations_data.txt
annotation1="hello"
annotation2="world"
kubectl.kubernetes.io/last-applied-configuration="{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{\"annotation1\":\"hello\",\"annotation2\":\"world\"},\"labels\":{\"app\":\"httpd\",\"cms\":\"wordpress\"},\"name\":\"pod-httpd\",\"namespace\":\"default\"},\"spec\":{\"containers\":[{\"image\":\"httpd:latest\",\"name\":\"cntr-httpd\",\"ports\":[{\"containerPort\":80}],\"volumeMounts\":[{\"mountPath\":\"/etc/pod_data\",\"name\":\"labels-and-annotations\"}]}],\"volumes\":[{\"downwardAPI\":{\"items\":[{\"fieldRef\":{\"fieldPath\":\"metadata.labels\"},\"path\":\"label_data.txt\"},{\"fieldRef\":{\"fieldPath\":\"metadata.annotations\"},\"path\":\"annotations_data.txt\"}]},\"name\":\"labels-and-annotations\"}]}}\n"
kubernetes.io/config.seen="2019-04-05T10:03:07.021217563Z"
kubernetes.io/config.source="api"root@pod-httpd:/etc/pod_data# cat labels_data.txt


root@pod-httpd:/etc/pod_data# cat labels_data.txt
app="httpd"
cms="wordpress"root@pod-httpd:/etc/pod_data# 

```

Notice, that the annotation file also include kubernetes internally generated annotations. 



## Modifying Labels and Annotations

You can create/update/delete pod labels/annotations after the pod has been created. 

```bash
$ kubectl label pod pod-httpd env=prod
pod/pod-httpd labeled

$ kubectl annotate pod pod-httpd farewell=goodbye
pod/pod-httpd annotated

$ kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          2m39s   172.17.0.7   minikube   <none>           <none>            app=httpd,cms=wordpress,env=prod
```

And the cool thing here is that the updates gets reflected in realtime inside the pod:

```bash
$ kubectl exec pod-httpd -c cntr-httpd -it -- bash
root@pod-httpd:/usr/local/apache2# cd /etc/pod_data

root@pod-httpd:/etc/pod_data# ls -l
total 0
lrwxrwxrwx 1 root root 27 Apr  5 10:13 annotations_data.txt -> ..data/annotations_data.txt
lrwxrwxrwx 1 root root 22 Apr  5 10:13 labels_data.txt -> ..data/labels_data.txt


root@pod-httpd:/etc/pod_data# cat annotations_data.txt
annotation1="hello"
annotation2="world"
farewell="goodbye"
...


root@pod-httpd:/etc/pod_data# cat labels_data.txt
app="httpd"
cms="wordpress"
env="prod"
```

There is another way that let's pods accessing even more data, and that is by using curl inside your pod to interact with your kube cluster's api. We'll cover that later. 

## References

[https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)
