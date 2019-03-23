# pod commands

The primary function of a docker container is to run a process. This process comes in the form of a command or an entrypoint script, and is baked into the docker image using the Dockerfile's [CMD](https://docs.docker.com/engine/reference/builder/#cmd) or the [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint).


These commands/script can be shortlived, if the container is supposed to perform a specific task, in which case the container stops once the commands/script finishes running. Containers also run continuously, becuase they provide an ongoing service, for example the httpd container provides an ongoing web service.


So if you build a pod using a docker image that runs a short lived process, for example:

```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
  labels:
    component: centos
spec: 
  containers:
    - name: cntr-centos
      image: centos
```

Then the pod is created, it runs it's command, then shuts down again within a few seconds.

```bash
$ kubectl apply -f configs/pod-centos-shortlived.yml
pod/pod-centos created
$ kubectl get pods
NAME         READY   STATUS              RESTARTS   AGE
pod-centos   0/1     ContainerCreating   0          2s
$ kubectl get pods
NAME         READY   STATUS      RESTARTS   AGE
pod-centos   0/1     Completed   0          5s
$ kubectl get pods
NAME         READY   STATUS             RESTARTS   AGE
pod-centos   0/1     CrashLoopBackOff   1          9s
$ kubectl get pods
NAME         READY   STATUS             RESTARTS   AGE
pod-centos   0/1     CrashLoopBackOff   1          12s
$ 
$ kubectl get pods
NAME         READY   STATUS             RESTARTS   AGE
pod-centos   0/1     CrashLoopBackOff   1          21s
$ kubectl get pods
NAME         READY   STATUS      RESTARTS   AGE
pod-centos   0/1     Completed   2          26s
$ 
```

Here the pods ran for less than a second before shutting down. However pods are supposed to be used for running continuous workloads, so when the container stopped, kubernetes thought something went wrong and tried to restart it:

```bash
$ kubectl describe pod pod-centos
...
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  15m                 default-scheduler  Successfully assigned default/pod-centos to minikube
  Normal   Pulling    14m (x4 over 15m)   kubelet, minikube  pulling image "centos"
  Normal   Pulled     14m (x4 over 15m)   kubelet, minikube  Successfully pulled image "centos"
  Normal   Created    14m (x4 over 15m)   kubelet, minikube  Created container
  Normal   Started    14m (x4 over 15m)   kubelet, minikube  Started container
  Warning  BackOff    33s (x70 over 15m)  kubelet, minikube  Back-off restarting failed container

```

Kubernetes will just keep restarting this in an endless cycle. and the pod will keep starting+stopping in an endless cycle too. To run containers that have shortlived workloads, you should run them as Kubernetes jobs or cronjobs objeect. We'll cover them later. 

If you want the container in this pod to run on an ongoing basis, then you can override the centos image's default CMD setting with an ongoing command/script using the command+args settings:

```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
  labels:
    component: centos
spec:
  containers:
    - name: cntr-centos
      image: centos
      command: ["/bin/bash", "-c"]       # this starts a bash terminal and feeds the args content into it
      args:                         # the args section here is used to store a small shell script
        - |
          while true ; do
            date
            sleep 10
          done
```

Here we're feeding an infinite while loop to keep the pod running continuously:

```bash
$ kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
pod-centos   1/1     Running   0          15s
```

We specified the 'date' command in the while loop, so you can monitor the pods standard output for this info:

```bash
$ kubectl logs pod-centos
Mon Mar 11 12:14:01 UTC 2019
Mon Mar 11 12:14:11 UTC 2019
Mon Mar 11 12:14:21 UTC 2019
Mon Mar 11 12:14:31 UTC 2019
Mon Mar 11 12:14:41 UTC 2019
Mon Mar 11 12:14:51 UTC 2019
Mon Mar 11 12:15:01 UTC 2019
Mon Mar 11 12:15:11 UTC 2019
Mon Mar 11 12:15:21 UTC 2019
Mon Mar 11 12:15:31 UTC 2019
Mon Mar 11 12:15:41 UTC 2019
Mon Mar 11 12:15:51 UTC 2019
```

You can also monitor the pods standard output in realtime by using the logs -f flag:

```bash
$ kubectl logs -f pod-centos
```

Or connect your bash terminal directly to the pod's standard output using the 'attach' command:

```bash
$ kubectl attach pod-centos
Defaulting container name to cntr-centos.
Use 'kubectl describe pod/ -n default' to see all of the containers in this pod.
If you don't see a command prompt, try pressing enter.
Mon Mar 11 12:21:51 UTC 2019
Mon Mar 11 12:22:01 UTC 2019
Mon Mar 11 12:22:11 UTC 2019
Mon Mar 11 12:22:21 UTC 2019
Mon Mar 11 12:22:31 UTC 2019
Mon Mar 11 12:22:41 UTC 2019
```

There could be times when you want to run commands/scripts in addition to the docker image's backed in CMD/Entrypoint, rather than over-riding it. Luckily there are other ways to inject commands/shellscripts into pods, using [Poststart/PreStop](https://kubernetes.io/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/) hooks. We'll cover them later. 