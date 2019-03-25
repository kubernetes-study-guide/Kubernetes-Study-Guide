# Quotas

Each worker node has a limited amount of cpu+ram available:

```bash
Capacity:
 cpu:                2
 ephemeral-storage:  16888216Ki
 hugepages-2Mi:      0
 memory:             2038624Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  15564179840
 hugepages-2Mi:      0
 memory:             1936224Ki
 pods:               110
```

So to make optimal use of these resources, we have to set quotas. 

# Pod Quotas

The containers pod might need a minimum amount of cpu+ram in order to work properly. If so then you can specify them with the `pod.spec.containers.resources.requests` settings. Also you can set the maximum amount of cpu+ram your pod's containers are allowed to use, with the `pod.spec.containers.resources.limits` setting, in case your container unexpected and starts using too much hardware resources which in turn could have a knock on impact on other things running on your kube cluster. 

```yaml
---
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-centos
  namespace: codingbee     # notice we are using a new namespace. This is in preperation for a later demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: centos_os
  template:
    metadata:
      labels:
        app: centos_os
    spec: 
      containers:
        - name: cntr-centos
          image: centos:latest
          resources:             # requests are used for minimum requirements. 
            requests:
              memory: "64Mi"
              cpu: "100m"    # 1000m = 1 cpu core. So here we're requesting just 10%.
            limits:
              memory: "128Mi"
              cpu: "200m"
          command: ["/bin/bash", "-c"]
          args:
            - |
              while true ; do
                date 
                sleep 10 
              done
          ports:
            - containerPort: 80
```



Since we're using a new namespace, codingbee, we'll switch over to that namespace for now:


```bash
$ kubectl config set-context  $(kubectl config current-context) --namespace=codingbee
Context "minikube" modified.

$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          default              kubernetes                   chowdhus             
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop   
*         minikube             minikube                     minikube             codingbee
```




The requests value is not necessarily how much of the resource that's actually used, it's just how much resource that should be reserved in case the pod needs it. the pod can however 

It's best practice to always include requests+limit setting in all pod definition, that's because it will help the kube-scheduler figure out which nodes to place the pods.

Note: If you exec into your containers, and then run `top` or `free -m`. Then you'll only see your worker node's cpu and ram. That's becuase these limits are applied to your pods externally by kubernetes. 


## Namespace Quota

It's best practice to always specify pod requests+limits for the optimal running of your kube cluster. You can force all pod definitions to require limit+requests settings, so that if they are omitted then kubernetes will throw error message. That's done by creating a ResourceQuota object. A resource quota is somethig that gets applied at the namespace level:


```yaml
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: codingbee-compute-quota
  namespace: codingbee
spec:
  hard:
    requests.cpu: 1
    limits.cpu: 2
    requests.memory: 1Gi
    limits.memory: 2Gi
```


You can also use ResourceQuota to set limits on the number of different types of Kubernetes objects that are allowed to exist in a namespace: 


```yaml
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: codingbee-object-quota
  namespace: codingbee
spec:
  hard:
    pods: 2
    secrets: 0
    services.nodeports: 0
    services.loadbalancers: 0
```

For examplem, here we're saying that you can't have more than 2 pods running inside the 'codingbee' namespace. After we apply these 2 ResourceQuotas, we end up with:

```bash
$ kubectl get ResourceQuota
NAME                      CREATED AT
codingbee-compute-quota   2019-03-20T18:20:25Z
codingbee-object-quota    2019-03-20T18:23:38Z


$ kubectl describe resourcequotas codingbee-compute-quota
Name:            codingbee-compute-quota
Namespace:       codingbee
Resource         Used   Hard
--------         ----   ----
limits.cpu       200m   2
limits.memory    128Mi  2Gi
requests.cpu     100m   1
requests.memory  64Mi   1Gi


$ kubectl describe resourcequotas codingbee-object-quota
Name:                   codingbee-object-quota
Namespace:              codingbee
Resource                Used  Hard
--------                ----  ----
pods                    1     2
secrets                 0     0
services.loadbalancers  0     0
services.nodeports      0     0
```

Objects limits is a powerful way to disable certain features in your namespace. E.g. If you want Terraform to be responsible for building loadbalancers, then you set services.loadbalancers to 0, and if you want Hashicorp Vault as your central secrets manager, then set secrets to 0. 

Let's now try to breach a limit, let's try to exeed the pod limit, by scaling our current deployment to 3 pods:

```bash
$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
dep-centos   1/1     1            1           18s

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
dep-centos-55989c5c59-vq78m   1/1     Running   0          20s



$ kubectl scale --current-replicas=1 --replicas=3 deployment dep-centos
deployment.extensions/dep-centos scaled

$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
dep-centos   2/3     2            2           53s

$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
dep-centos-55989c5c59-rx6p6   1/1     Running   0          8s
dep-centos-55989c5c59-vq78m   1/1     Running   0          56s
```

So it looks like only 2 pods are created when in fact we've requested for 3. So it looks like the replicaset is struggling to create all the pods. If you take a look at the replicasets events, you'll see why:

```bash
$ kubectl describe rs dep-centos-55989c5c59
...
Conditions:
  Type             Status  Reason
  ----             ------  ------
  ReplicaFailure   True    FailedCreate
Events:
  Type     Reason            Age                  From                   Message
  ----     ------            ----                 ----                   -------
  Normal   SuccessfulCreate  10m                  replicaset-controller  Created pod: dep-centos-55989c5c59-vq78m
  Normal   SuccessfulCreate  10m                  replicaset-controller  Created pod: dep-centos-55989c5c59-rx6p6
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-724dd" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-dhtbc" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-6zscd" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-mgt7k" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-78d9x" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-whp6h" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-2wscl" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-69tbg" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      10m                  replicaset-controller  Error creating: pods "dep-centos-55989c5c59-pqldz" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
  Warning  FailedCreate      4m57s (x8 over 10m)  replicaset-controller  (combined from similar events): Error creating: pods "dep-centos-55989c5c59-drsmm" is forbidden: exceeded quota: codingbee-object-quota, requested: pods=1, used: pods=2, limited: pods=2
```




## LimitRange

When you have ResourceQuota object with limits in place for cpu+ram, it meant that it has become mandatory for all pods definitons to specify cpu+ram limits as well. So the following will no longer work:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: apache_webserver
spec:
  containers:
    - name: cntr-httpd
      image: httpd:latest
      ports:
        - containerPort: 80
```

Trying to apply this will now fail:

```bash
$ kubectl apply -f configs/pod-httpd.yml
Error from server (Forbidden): error when creating "configs/pod-httpd.yml": pods "pod-httpd" is forbidden: failed quota: codingbee-compute-quota: must specify limits.cpu,limits.memory,requests.cpu,requests.memory
```

So this presents 2 problems:

- You might have lots of existing yaml files that now needs to be updated
- What if definitions are set too excessive, e.g. requests.cpu is set something ridiculously high is 10. 

To address this you need to create a LimitRange object. 

```yaml
---
apiVersion: v1
kind: LimitRange
metadata:
  name: limitrange-codingbee
  namespace: codingbee
spec:
  limits:
    - type: pod      # This sets pod level quotas, i.e. the total of all containers in a pod
      min:
        cpu: 50m
        memory: 10Mi
      max:
        cpu: 800m
        memory: 200Mi
    - type: container    # This sets limits and default quotas at the pod level
      min:
        cpu: 50m
        memory: 10Mi
      max:
        cpu: 400m
        memory: 200Mi
      defaultRequest:     # default requests value
        cpu: 100m
        memory: 50Mi
      default:            # default limit values
        cpu: 200m
        memory: 100Mi
        
```

After applying this, you'll end up with:

```bash
$ kubectl get limitranges
NAME                   CREATED AT
limitrange-codingbee   2019-03-22T15:57:02Z


$ kubectl describe limitranges limitrange-codingbee
Name:       limitrange-codingbee
Namespace:  codingbee
Type        Resource  Min   Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---    ---------------  -------------  -----------------------
Pod         cpu       50m   800m   -                -              -
Pod         memory    10Mi  200Mi  -                -              -
Container   memory    10Mi  200Mi  50Mi             100Mi          -
Container   cpu       50m   400m   100m             200m           -
```

You'll now be able to keep using your pod defs, and defaults are used:


```bash
$ kubectl describe pod pod-httpd
...
    Limits:
      cpu:     200m
      memory:  100Mi
    Requests:
      cpu:        100m
      memory:     50Mi
...
```











