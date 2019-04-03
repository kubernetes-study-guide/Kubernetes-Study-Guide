# hostPath volumes

hostPath volumes let's you mount one of your worker node's directory onto your pod's containers. Here's an example:


```yaml 
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: apache_webserver
spec:
  volumes:
    - name: webcontent    # here we specify the name of the volume
      hostPath:
        path: /tmp/webcontent  # this folder will get created inside minikube vm if it doesn't already exist. 
        type: DirectoryOrCreate  # this is one of several options at https://kubernetes.io/docs/concepts/storage/volumes/#hostpath
  containers:
    - name: cntr-httpd
      image: httpd
      volumeMounts:
        - name: webcontent
          mountPath: /usr/local/apache2/htdocs
      ports:
        - containerPort: 80
```

Once you have applied this, you should see the following folder now exists (you can also created some test content in this folder):

```bash
$ minikube ssh
$ ls -l /tmp | grep webcontent
drwxr-xr-x 2 root root   40 Mar  7 19:24 webcontent
$ echo 'hello hostpath content' > /tmp/webcontent/index.html
```

Then back on our macbook we apply and test:

```bash
$ curl http://192.168.99.100:31000
hello hostpath content
```


You can share the same hostpath directory across multiple containers/pods on the same worker node. One possible usecase for this is that you can store all logs centrally using hostPaths, and then use DeamonSet to deploy a single filebeat pod, which ships all the hostPaths content to an elk server. This would eliminate the need to have a filebeat sidecar container for each pod. 

Some key points about hostPath volumes:

- if worker node dies, then all the hostPath volume stored data is lost with it. 
- if pod dies and gets rebuilt, then there's a good chance it will get rebuilt on a different worker node. A way round this problem is to build the pods using stateful states controller objects.
- Like emptyDir, the underlying storage that hostPath uses is the worker node's diskspace.


So to summarise:

emptyDir volumes stores data external to container, but inside containers, hostPath volumes stores data external to pods, but inside worker node. 