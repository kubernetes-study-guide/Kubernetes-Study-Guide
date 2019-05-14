# pod commands

The primary function of a docker container is to run an executable (e.g. a binary, command, or shell script) along with some optional arguments. The executable+arguments are both specified in the Dockerfile with the following settings:

- [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)
- [CMD](https://docs.docker.com/engine/reference/builder/#cmd) - There's three ways to use this setting - the 2nd approach, which is to use CMD in conjunction with the ENTRYPOINT is the recommended way. This may look untuitively but it ensures that the primary process (PID 1) inside the container isn't the bash/sh session wrapper, but is the primary executable.

This entrypoint's binary/command/script can be used to run:

- **shortlived workloads** - if the container is supposed to perform a specific task.
- **ongoing workloads** - E.g. running the apache httpd binary to provide an ongoing web service.

## Shortlived Workloads (eg1-shortlived)

Pods are designed for running containers with ongoing workloads. However, if your pod is built from an image, whose entrypoint is a shortlived (e.g. centos):

```yaml
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
$ kubectl get pods
NAME         READY   STATUS             RESTARTS   AGE
pod-centos   0/1     CrashLoopBackOff   1          21s
$ kubectl get pods
NAME         READY   STATUS      RESTARTS   AGE
pod-centos   0/1     Completed   2          26s
$
```

Here the pods ran for less than a second before shutting down, kubernetes thought something went wrong and restarted the container, and keeps restarting it in an endless cycle:

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

To run containers that have shortlived workloads, you should run them as Kubernetes a **jobs** or **cronjobs** object. We'll cover them later.

## Ongoing Workloads (eg2-ongoing)

Lets say you still want to use the centos image for the primary container in your pod. That's still possible, by overriding the centos image's default ENTRYPOINT/CMD, with an ongoing command/script using the command+args settings:

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

Here we used the following settings:

- `pod.spec.containers.command` - This overrides/adds the docker images 'ENTRYPOINT' setting.
- `pod.spec.containers.args` - This override/adds the docker images 'CMD' setting

Here we're feeding an infinite while loop to keep the pod running continuously:

```bash
$ kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
pod-centos   1/1     Running   0          15s
```

We specified the 'date' command in the while loop, so you can monitor the pods standard output for this info:

```bash
$ kubectl logs pod-centos -c cntr-centos
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
kubectl logs -f pod-centos -c cntr-centos
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

In this demo we use an image with an shortlived workload. However you can use this approach to replace a ongoing workload with another ongoing workload.

## Other workloads

There could be times when you want to run commands/scripts in addition to the docker image's CMD/Entrypoint, rather than over-riding it. Luckily there are other ways to inject commands/shellscripts into pods, using [Poststart/PreStop](https://kubernetes.io/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/) hooks. We'll cover them later.

You can also run non-primary containers with shortlived workloads using `pod.spec.initContainers`, which will cover later.

You can also run other commands that periodically monitors the health of your pod's containers. These are known as [liveness and readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/). We'll cover these later too.
