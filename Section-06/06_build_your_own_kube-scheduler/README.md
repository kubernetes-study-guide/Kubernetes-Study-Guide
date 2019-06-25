# Build your own Kube-Scheduler

In (a minikube or) a kubeadm provisioned kube cluster, the kube-scheduler comes in the form of a pod:

```bash
root@kube-master:~# kubectl get pods --namespace=kube-system -o wide | grep scheduler
kube-scheduler-kube-master            1/1     Running   1          42h   10.2.5.110    kube-master    <none>           <none>
```

This pod makes use of the following image:


```bash
root@kube-master:~# kubectl describe pod kube-scheduler-kube-master --namespace=kube-system
Name:               kube-scheduler-kube-master
Namespace:          kube-system
Priority:           2000000000
PriorityClassName:  system-cluster-critical
Node:               kube-master/10.2.5.110
Start Time:         Sat, 13 Apr 2019 11:04:57 +0000
Labels:             component=kube-scheduler
                    tier=control-plane
Annotations:        kubernetes.io/config.hash: f44110a0ca540009109bfc32a7eb0baa
                    kubernetes.io/config.mirror: f44110a0ca540009109bfc32a7eb0baa
                    kubernetes.io/config.seen: 2019-04-11T20:21:55.751863137Z
                    kubernetes.io/config.source: file
Status:             Running
IP:                 10.2.5.110
Containers:
  kube-scheduler:
    Container ID:  docker://61990e5882d9cfcea5a3f35423d2414d442caefb30fb9aa2e6491dc103b4640a
    Image:         k8s.gcr.io/kube-scheduler:v1.14.1
    Image ID:      docker-pullable://k8s.gcr.io/kube-scheduler@sha256:11af0ae34bc63cdc78b8bd3256dff1ba96bf2eee4849912047dee3e420b52f8f
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-scheduler
      --bind-address=127.0.0.1
      --kubeconfig=/etc/kubernetes/scheduler.conf
      --leader-elect=true
    State:          Running
      Started:      Sat, 13 Apr 2019 11:04:59 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Thu, 11 Apr 2019 20:21:57 +0000
      Finished:     Fri, 12 Apr 2019 22:30:14 +0000
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:        100m
    Liveness:     http-get http://127.0.0.1:10251/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/kubernetes/scheduler.conf from kubeconfig (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/scheduler.conf
    HostPathType:  FileOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:            <none>
```

This image is essentially an generic image that has the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) binary. This means it's quite easy to [create your own kube-scheduler image, followed by kube-scheduler pod](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/).

Once you have created your kubescheduler pod, you can then make use of it by using the `pod.spec.schedulerName` in your yaml descriptors. 




## References

[https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)