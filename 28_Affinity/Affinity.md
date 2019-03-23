# Affinity and Anti-Affinity

We came across nodeSelector as part of looking at deamonsets. [Affinity/Anti-Affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) can do the same thing as a nodeSelector but also has more capabilities:

1. More versatile label selection method - e.g. instead of a simple key=value label match. You can just specify deploy/dont-deploy on nodes that has a key with a certain and ignoring what the key's value is. Or you can specify a list of valid values for a given key.  See `pod.spec.affinity.nodeAffinity`
2. Specify preference (rather than hard rules) - So if no suitable deployment target is found, kubernetes will deploy it anyway to non-mathing targets, since it's more important for the pod(s) to exist than having those pods running on non-preferred worker, see `pod.spec.affinity.podAffinity.preferredDuringSchedulingIgnoredDuringExecution`
3. Prevent particular pods from running on the same worker node (co-locating), based on labels. See `pod.spec.affinity.podAntiAffinity`.


## NodeAffinity Preference (eg1-node-affinity)

For this demo we'll create the following node label:


```bash
$ kubectl label nodes kube-worker2 ec2InstanceType=M3
node/kube-worker2 labeled

$ kubectl get nodes --show-labels
NAME           STATUS   ROLES    AGE   VERSION   LABELS
kube-master    Ready    master   81m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-master,node-role.kubernetes.io/master=
kube-worker1   Ready    <none>   77m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-worker1
kube-worker2   Ready    <none>   75m   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ec2InstanceType=M3,kubernetes.io/hostname=kube-worker2

```

then our affinity setting is going to be:


```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: apache_webserver
spec:
  affinity:                # Added this section
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution: # this means decision is made at scheduling stage only, so won't self correct if rule is met in the future
        - weight: 10
          preference:
            matchExpressions: 
              - key: ec2InstanceType
                operator: In
                values:
                  - M1
                  - M2
  containers:
    - name: cntr-httpd
      image: httpd:latest
      ports:
        - containerPort: 80
```


Notice here that our Kube-worker1 node is the closest match with label value is M3, but yaml file will match for either M1 or M2. So no match is made. However a pod is still created, since it's more important for the pod(s) to exist on non-ideal worker nodes, than not have any pod(s) at all. 

```bash
# kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          9m59s   192.168.1.3   kube-worker1   <none>           <none>            app=apache_webserver
```

If a match subsequently came to existance, e.g. matching label applied to existing worker, or new worker node is added to the cluster with matching label, then nothing will happen as implied by 'preferredDuringSchedulingIgnoredDuringExecution'.

```bash
$ kubectl label nodes kube-worker2 ec2InstanceType=M2 --overwrite
node/kube-worker2 labeled

$ kubectl get nodes -o wide --show-labels
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME   LABELS
kube-master    Ready    master   3h51m   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-master,node-role.kubernetes.io/master=
kube-worker1   Ready    <none>   3h48m   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-worker1
kube-worker2   Ready    <none>   3h46m   v1.13.4   10.0.2.15     <none>        Ubuntu 16.04.5 LTS   4.4.0-131-generic   docker://18.6.1     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ec2InstanceType=M2,kubernetes.io/hostname=kube-worker2

$ kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          24m   192.168.1.3   kube-worker1   <none>           <none>            app=apache_webserver

```


Even reapplying wont make a difference since the pod already exists:


```bash
$ kubectl apply -f configs/eg1-node-affinity/
pod/pod-httpd unchanged


$ kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          25m   192.168.1.3   kube-worker1   <none>           <none>            app=apache_webserver
```

The only way to fix this is by rebuilding the pod:


```bash
# kubectl delete -f configs/eg1-node-affinity/ ; kubectl apply -f configs/eg1-node-affinity/
pod "pod-httpd" deleted
pod/pod-httpd created
# kubectl get pods -o wide --show-labels
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
pod-httpd   1/1     Running   0          7s    192.168.2.4   kube-worker2   <none>           <none>            app=apache_webserver
```

If you have created a pod cluster via a controller, e.g. deployment, then you can trigger a rebuild by manually deleting one pod at a time.


### preferredDuringSchedulingIgnoredDuringExecution weights

The 'weight' setting is something specific to soft/preference rules. For each preference rule a node matches, it receives a score equal to the weight. So if a certain node matches multiple preferences then it scores higher, and is more likely to have the pods deployed to them.

## Hard rules (eg2-hard-node-affinity)

We just saw a soft rule (preference) in action. But if you increased the replica to 2, then it gets deployed on matching and non matching worker nodes. That's because we only have 2 worker nodes and pods needs to be on both for HA. I.e. need for HA overrides the soft rule. If you want more powerful rules, the you need to make use of hard rules. 

For hard rules you use `pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution` . This does the same job as nodeSelector but with more advanced customisation. The syntax also looks similar to soft rules:

```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dep-httpd
  labels:
    app: apache_webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      component: httpd
  template:
    metadata:
      labels:
        component: httpd
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:   # notice, no weight setting. 
            nodeSelectorTerms:                                  # notice 'preference' replaced by 'nodeSelectorTerms'
              -  matchExpressions: 
                   - key: ec2InstanceType
                     operator: In
                     values:
                       - M1
                       - M2
      containers:
        - name: cntr-httpd
          image: httpd:latest
          ports:
            - containerPort: 80
```

This results in:

```bash
$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
dep-httpd-69779b8c84-9jrw8   1/1     Running   0          5m52s   192.168.2.9   kube-worker2   <none>           <none>
dep-httpd-69779b8c84-cjmnt   1/1     Running   0          5m52s   192.168.2.8   kube-worker2   <none>           <none>
```

I.e. both nodes are on the same worker node, even though there is another worker node available.




## Built-in node labels

If you take a look at the node labels again:

```bash
# kubectl get nodes --show-labels
NAME           STATUS   ROLES    AGE   VERSION   LABELS
kube-master    Ready    master   16h   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-master,node-role.kubernetes.io/master=
kube-worker1   Ready    <none>   16h   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=kube-worker1
kube-worker2   Ready    <none>   16h   v1.13.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ec2InstanceType=M2,kubernetes.io/hostname=kube-worker2
```

You'll see that the nodes are already tagged with a few labels by default. You can use these built-in labels as part of your Affinity/Anti-Affinity definitions if they meet your needs. These builtin node labels are often used in conjunction with podAffinity


## podaffinity

Sometimes you might want 2 or more different pods running on the same worker node. For example you might want your wordpress pod and the wordpress's mysql pod running on the same worker, in order to reduce network latency when your wordpress pod talks to the mysql pod. This can be done using the `pod.spec.affinity.podAffinity` setting. This setting can be used to co-locate pods in the same (aws availabity) zone as well as node.

So to take a look at this, lets start by creating a pod that we want to have other pods to have affinity with:

```bash
$ kubectl apply -f configs/eg3-podaffinity/pod-mysql.yml
pod/pod-mysql-db created
$ kubectl get pods -o wide --show-labels
NAME           READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
pod-mysql-db   1/1     Running   0          18s   192.168.1.6   kube-worker1   <none>           <none>            component=mysql_db_server
```

Now we want our pods to have affinity with this pod, and we create the connection using the component=mysql_db_server pod label:

```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dep-httpd
  labels:
    app: apache_webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      component: httpd
  template:
    metadata:
      labels:
        component: httpd
    spec:
      affinity:
        podAffinity:                                       # here we set the podaffinity setting
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  component: mysql_db_server
              topologyKey: kubernetes.io/hostname        # This is used to target a single or group of nodes
      containers:
        - name: cntr-httpd
          image: httpd:latest
          ports:
            - containerPort: 80
```


The 'matchLabels' section is used to identify which pod you want your pod to be co-located with, the topologyKey setting specifies to which extent you want to what extent you want the co-location to be. In our case we used one of the builtin node labels to specify that we want to co-locate on the exact same node (based on hostname) that the mysql pod is currently on. If there was a 'zone' label available then we could have specified all nodes in the same AZ. 

So applying this results in:

```bash
$ kubectl get pods -o wide --show-labels
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
dep-httpd-5b85b48f69-czd7p   1/1     Running   0          16s   192.168.1.7   kube-worker1   <none>           <none>            component=httpd,pod-template-hash=5b85b48f69
dep-httpd-5b85b48f69-t4fp2   1/1     Running   0          16s   192.168.1.8   kube-worker1   <none>           <none>            component=httpd,pod-template-hash=5b85b48f69
pod-mysql-db                 1/1     Running   0          52m   192.168.1.6   kube-worker1   <none>           <none>            component=mysql_db_server
```

so both httpd pods ends up on the same node as the mysql pod.



## podAntiAffinity

podAntiAffinity is the reverse of podaffinity, i.e. deploy your pods away from another set of pods. 

```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dep-httpd
  labels:
    app: apache_webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      component: httpd
  template:
    metadata:
      labels:
        component: httpd
    spec:
      affinity:
        podAntiAffinity:                                       # Just need to change this line to do the reverse
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  component: mysql_db_server
              topologyKey: kubernetes.io/hostname
      containers:
        - name: cntr-httpd
          image: httpd:latest
          ports:
            - containerPort: 80
```

Applying this results in:


```bash
$ kubectl get pods -o wide --show-labels
NAME                        READY   STATUS              RESTARTS   AGE    IP            NODE           NOMINATED NODE   READINESS GATES   LABELS
dep-httpd-586fd6f97-9kz5g   0/1     ContainerCreating   0          2s     <none>        kube-worker2   <none>           <none>            component=httpd,pod-template-hash=586fd6f97
dep-httpd-586fd6f97-rbvvc   0/1     ContainerCreating   0          2s     <none>        kube-worker2   <none>           <none>            component=httpd,pod-template-hash=586fd6f97
pod-mysql-db                1/1     Running             0          111m   192.168.1.6   kube-worker1   <none>           <none>            component=mysql_db_server

```
