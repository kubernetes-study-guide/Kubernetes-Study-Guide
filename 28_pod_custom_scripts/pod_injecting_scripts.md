# running custom commands/scripts inside your pod


Earlier we saw how we can run custom commands in a pod's container by using the pod.spec.containers.command setting, which effectively overrides the image's default command. However there could be times when you want to run commands/scripts in addition to the docker image's baked-in CMD/Entrypoint scripts, rather than overriding it. That's possible by using the pod.spec.containers.lifecycle.postStart and pod.spec.containers.lifecycle.postStart and pod.spec.containers.lifecycle.preStop settings, aka [Poststart/PreStop hooks](https://kubernetes.io/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/).


## postStart

postStart is used for running scripts during a container's launch time. postStart scripts runs in parrallel to the image's default CMD/Entrypoint. postStart scripts are added under the pod.spec.containers.lifecycle.postStart.exec yaml setting:


```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: httpd
spec:
  containers:
    - name: cntr-httpd
      image: httpd
      lifecycle:          # we add in the lifecycle section which houses the 
        postStart:             # podStart section
          exec:
            command:
              - /bin/bash
              - -c
              - |
                echo "the date is: $(date)" > /usr/local/apache2/htdocs/index.html
                echo 'This content was created by the postStart hook' >> /usr/local/apache2/htdocs/index.html
      ports:
        - containerPort: 80
```


# preStop

There could be various tasks (e.g. cleanup tasks) that you may want to run as part of terminating a pod. That's possible using the pod.spec.containers.lifecycle.preStop.exec yaml setting:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: httpd
spec:
  volumes:
    - name: list-of-deleted-pods
      hostPath:
        path: /tmp/ListOfDeletedPods
        type: DirectoryOrCreate
  containers:
    - name: cntr-httpd
      image: httpd
      lifecycle:
        postStart:
          exec:
            command:
              - /bin/bash
              - -c
              - |
                echo "the data is $(date)" > /usr/local/apache2/htdocs/index.html
                echo 'this content was created by the postStart hook'  >> /usr/local/apache2/htdocs/index.html
        preStop:
          exec:
            command:
              - /bin/bash
              - -c
              - |
                echo "the date is $(date)" > /preStopScriptOutput/$(hostname).txt
                echo "the pod $(hostname) has been terminated" >> /preStopScriptOutput/$(hostname).txt
      volumeMounts:
        - name: list-of-deleted-pods
          mountPath: /preStopScriptOutput
      ports:
        - containerPort: 80
```

It's a little bit trickier to check to see if the preStoop script has worked since you can't investigate the pod after it's been deleted. So in our case we have created a volumeMount that the preStop script will drop a 'flag file' as part of it's termination script. So let's create this:

```bash
$ kubectl apply -f configs/eg2-PreStop
pod/pod-httpd created
service/svc-nodeport-httpd created


$ curl $(minikube service svc-nodeport-httpd --url)
the data is Tue Mar 12 09:12:00 UTC 2019
this content was created by the postStart hook
```

Here we can see that the preStart script has executed ok, at 9:12am. I wait a couple of minutes and then delete the pod:

```bash
$ kubectl delete -f configs/eg2-PreStop
pod "pod-httpd" deleted
service "svc-nodeport-httpd" deleted
```

Now let's check if the termination created the flag file:

```bash
$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)


$ cd /tmp/ListOfDeletedPods/

$ ls -l
total 4
-rw-r--r-- 1 root root 79 Mar 12 09:15 pod-httpd.txt

$ cat pod-httpd.txt
the date is Tue Mar 12 09:15:35 UTC 2019
the pod pod-httpd has been terminated
```

Although this is just a demo, a possible usecase for this kind of setup, is to have a single pod per node (setup via daemon set) which specialise in performing cleanup task. This pod can be used to constantly poll for the hostPath for new pod deletion and then perform cleanup tasks associated with thos pods. 

## initContainers

In a pod, you can have multiple containers, a primary container, and optionally 1 or more secondary (sidecar) containers. Howevever there's another type of container can add in, called [initContainers](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/). initContainers (aka initialisation containers) are shortlived containers that are launched at a pod's launch time, as soon as the initContainers stops, the main primary container is then started up.

One possible usecase for this is to use it populate a emptyDir non-persistant volume with website files, e.g. by git cloning a repo. Then the primary httpd container can mount that volume and display it. You can setup initcontianers using the pod.spec.initContainers yaml setting:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: httpd
spec:
  initContainers:      # here we'ved added the initialisation contaner
    - name: init
      image: centos
      command:
        - /bin/bash
        - -c
        - |
          echo '$(date) INFO: sleep for 1 minute'
          sleep 60
  containers:
    - name: cntr-httpd
      image: httpd
      ports:
        - containerPort: 80
```


Straight after applying the above we get the following output:

```bash
$ kubectl apply -f configs/eg3-initContainers
pod/pod-httpd created
service/svc-nodeport-httpd created
$ kubectl get pod
NAME        READY   STATUS     RESTARTS   AGE
pod-httpd   0/1     Init:0/1   0          30s
```

Notice that the output is a little different to indicate that the init container is currently running. Also if you try to check logs:

```bash
$ kubectl logs pod-httpd
Error from server (BadRequest): container "cntr-httpd" in pod "pod-httpd" is waiting to start: PodInitializing
$ kubectl logs pod-httpd -c init
INFO: sleep for 1 minute
```

Our primary container usually starts up in well under 30 seconds, but in this case we have stalled the primary container launch using the init container, as a demo:

```bash
$ kubectl get pod
NAME        READY   STATUS     RESTARTS   AGE
pod-httpd   0/1     Init:0/1   0          55s
$ curl $(minikube service svc-nodeport-httpd --url)
curl: (7) Failed to connect to 192.168.99.105 port 31000: Connection refused
```

The primary container only starts working a few seconds after the initcontainers finishes running:

```bash
$ kubectl get pod

NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          78s

$ curl $(minikube service svc-nodeport-httpd --url)
<html><body><h1>It works!</h1></body></html>
```



