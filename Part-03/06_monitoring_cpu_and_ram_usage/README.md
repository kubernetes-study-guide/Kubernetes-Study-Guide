# Monitoring cpu and ram

In this article we're going to demo how you can view how much ram+cpu is being used by your nodes, pods, and containers. You can get these metrics using the 'top' command:

```bash
$ kubectl top nodes
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```

However the monitoring capabilities doesn't come include with a kubernetes installm which is why we get the above error message. You need to install add this capability in by installing the [metrics-server](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server). On a minikube, you just have to enable it:

```bash
$ minikube addons enable metrics-server
✅  metrics-server was successfully enabled
```

Otherwise you would need to run:

```bash
$ git clone https://github.com/kubernetes-incubator/metrics-server.git
Cloning into 'metrics-server'...
remote: Enumerating objects: 1467, done.
remote: Counting objects: 100% (1467/1467), done.
remote: Compressing objects: 100% (1021/1021), done.
remote: Total 11096 (delta 467), reused 1040 (delta 340), pack-reused 9629
Receiving objects: 100% (11096/11096), 13.08 MiB | 2.11 MiB/s, done.
Resolving deltas: 100% (5338/5338), done.
Checking out files: 100% (2967/2967), done.

$ cd metrics-server/

$ kubectl apply -f metrics-server/deploy/1.8+/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.extensions/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

Notice that this repo is the 'kubernetes-incubator' github project, which means that this metric-server project is still a work in progress. I think that this also means that it's unlikely you will be asked to install metrics-server in the CKA exam. Instead it will be already installed for you.

After this install, you should now get:

```bash
$ kubectl top pods
error: metrics not available yet
$ kubectl top pods
error: metrics not available yet
```

Now you need to wait a few minutes.

```bash
$ kubectl top nodes
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
minikube   163m         8%     1448Mi          76%

$ kubectl top pods
NAME                     CPU(cores)   MEMORY(bytes)
pod-alpine-default       0m           0Mi
pod-alpine-runasuser     0m           0Mi
pod-capabilities         0m           0Mi
pod-fsgroup              0m           7Mi
pod-privileged           0m           0Mi
pod-readonlyrootfs       0m           0Mi
pod-runasnonroot-guest   0m           0Mi
```

To see how much cpu+mem is being used by kubernetes itself:

```bash
$ kubectl top pods --namespace=kube-system
NAME                                        CPU(cores)   MEMORY(bytes)
coredns-fb8b8dccf-hq87w                     2m           17Mi  
coredns-fb8b8dccf-rm7b7                     2m           7Mi
default-http-backend-6864bbb7db-6t24l       0m           1Mi
etcd-minikube                               18m          30Mi
kube-addon-manager-minikube                 11m          4Mi
kube-apiserver-minikube                     28m          209Mi
kube-controller-manager-minikube            12m          44Mi
kube-proxy-vj825                            1m           13Mi
kube-scheduler-minikube                     1m           10Mi
kubernetes-dashboard-79dd6bfc48-fpq52       0m           18Mi
metrics-server-77fddcc57b-cvvnx             0m           20Mi
nginx-ingress-controller-586cdc477c-nkrz4   3m           79Mi
storage-provisioner                         0m           31Mi
```

If a pod has multiple containers then you can also see how much cpu+ram each container is using:

```bash
$ kubectl get pods
NAME                      READY   STATUS                       RESTARTS   AGE
pod-alpine-default        1/1     Running                      0          7m33s
pod-alpine-runasnonroot   0/1     CreateContainerConfigError   0          7m33s
pod-alpine-runasuser      1/1     Running                      0          7m33s
pod-capabilities          1/1     Running                      0          7m33s
pod-fsgroup               2/2     Running                      0          7m33s  # this pod has 2 containers
pod-privileged            1/1     Running                      0          7m33s
pod-readonlyrootfs        1/1     Running                      0          7m33s
pod-runasnonroot-guest    1/1     Running                      0          7m33s


$ kubectl top pods pod-fsgroup
NAME          CPU(cores)   MEMORY(bytes)
pod-fsgroup   0m           7Mi

$ kubectl top pods pod-fsgroup --containers
POD           NAME          CPU(cores)   MEMORY(bytes)
pod-fsgroup   cntr-httpd    0m           6Mi
pod-fsgroup   cntr-centos   0m           1Mi
```
