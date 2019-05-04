# Pods accessing it's own Metadata

With pods you can inject a pods metadata into a pod in the form of a container environment variables. This is done using the `pod.spec.containers.env.valueFrom.fieldRef` feature:

```
$ kubectl explain pod.spec.containers.env.valueFrom
...
fieldRef     <Object>
     Selects a field of the pod: supports metadata.name, metadata.namespace,
     metadata.labels, metadata.annotations, spec.nodeName,
     spec.serviceAccountName, status.hostIP, status.podIP.
...
```
Note: metadata.labels and metadata.annotations, doesn't work. I think this could be a typo. 


You can inject all these metadata into pods like this:


```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  annotations:
    annotation1: hello
  labels:
    app: httpd
spec:
  containers:
    - name: cntr-httpd
      image: httpd:latest
      ports:
        - containerPort: 80
      env:  
        - name: pod_name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: pod_namespace
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: pod_nodeName
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: pod_serviceAccountName
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: pod_hostIP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        - name: pod_podIP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```

After creating this pod, and taking a look inside you'll find:


```bash
$ kubectl exec pod-httpd -c cntr-httpd -it bashroot@pod-httpd:/usr/local/apache2# env | grep '^pod'
pod_podIP=172.17.0.7
pod_nodeName=minikube
pod_namespace=default
pod_hostIP=10.0.2.15
pod_serviceAccountName=default
pod_name=pod-httpd
```

If you want to access other data that's not listed above, then you can do it by other means, such as 


## References
[inject pod data into containers](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/).



