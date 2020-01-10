# security contexts

Securtiy Contexts are a number of security related settings that you can set in your pods. You set Security contexts at the container level `pod.spec.securityContext`: 

```bash
$ kubectl explain pod.spec.containers.securityContext --recursive
...
FIELDS:
   allowPrivilegeEscalation     <boolean>
   capabilities <Object>
      add       <[]string>
      drop      <[]string>
   privileged   <boolean>
   procMount    <string>
   readOnlyRootFilesystem       <boolean>
   runAsGroup   <integer>
   runAsNonRoot <boolean>
   runAsUser    <integer>
   seLinuxOptions       <Object>
      level     <string>
      role      <string>
      type      <string>
      user      <string>
```

Some security contexts can be set at the pod level too:

```bash
$ kubectl explain pod.spec.securityContext --recursive
...
FIELDS:
   fsGroup      <integer>
   runAsGroup   <integer>
   runAsNonRoot <boolean>
   runAsUser    <integer>
   seLinuxOptions       <Object>
      level     <string>
      role      <string>
      type      <string>
      user      <string>
   supplementalGroups   <[]integer>
   sysctls      <[]Object>
      name      <string>
      value     <string>      
```

In our demo we created the following containers:

```bash
$ kubectl get pods
NAME                      READY   STATUS                       RESTARTS   AGE
pod-alpine-default        1/1     Running                      0          3h32m
pod-alpine-runasnonroot   0/1     CreateContainerConfigError   0          3h32m
pod-alpine-runasuser      1/1     Running                      0          3h32m
pod-capabilities          1/1     Running                      0          3h21m
pod-privileged            1/1     Running                      0          3h32m
pod-readonlyrootfs        1/1     Running                      0          9s
pod-runasnonroot-guest    1/1     Running                      0          3h32m
```

pod-alpine-default is a pod without any Security contexts. 

## runAsUser demo

By default, everything inside a container runs as the root user:

```bash
$ kubectl exec pod-alpine-default -- id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

$ kubectl exec pod-alpine-default -- whoami
root
```

However we can change this default user to a different user using the runAsUser setting:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-alpine-runasuser
spec:
  containers:
    - name: cntr-alpine
      image: alpine
      securityContext:                # we add this section.
        runAsUser: 405             # this value depends on the image you use. 
      command: ["sh", "-c"]
      args:
        - |
          while true ; do
            date
            id 
            whoami
            sleep 10 
          done
```

This result in:


```bash
$ kubectl exec pod-alpine-runasuser -- id
uid=405(guest) gid=100(users)

$ kubectl exec pod-alpine-runasuser -- whoami
guest

$ kubectl log pod-alpine-runasuser 
log is DEPRECATED and will be removed in a future version. Use logs instead.
Fri Apr 19 11:21:11 UTC 2019
uid=405(guest) gid=100(users)
guest
```

Note, we used the user id '405' because the alpine image has that guest user baked in:

```bash
$ kubectl exec pod-alpine-runasuser -- cat /etc/passwd | grep guest
guest:x:405:100:guest:/dev/null:/sbin/nologin
```
So you need to look in your images `/etc/passwd` to see what available users are there.



## runAsNonRoot

The default user that runs workloads inside the container is baked into the image during the image's build time, using the setting's specified in the image's dockerfile. If there's an image you use that by default run workloads as a non-root user, then as a precautionary measure you can also add in the following: 

```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-alpine-runasnonroot
spec:
  containers:
    - name: cntr-alpine
      image: alpine
      securityContext:
        runAsNonRoot: true           # we set this boolean
      command: ["sh", "-c"]
      args:
        - |
          while true ; do
            date
            id 
            whoami
            sleep 10 
          done
```


This means that if a future version of the image changes the default user from nonroot to root user, then you want kubernetes to refuse to deploy it:

This fails:


```bash
 kubectl get pods pod-alpine-runasnonroot
NAME                      READY   STATUS                       RESTARTS   AGE
pod-alpine-runasnonroot   0/1     CreateContainerConfigError   0          12m

$ kubectl describe pods pod-alpine-runasnonroot
...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  13m                  default-scheduler  Successfully assigned default/pod-alpine-runasnonroot to minikube
  Normal   Pulled     11m (x8 over 13m)    kubelet, minikube  Successfully pulled image "alpine"
  Warning  Failed     11m (x8 over 13m)    kubelet, minikube  Error: container has runAsNonRoot and image will run as root
  Normal   Pulling    3m2s (x41 over 13m)  kubelet, minikube  Pulling image "alpine"
```


You can specify both 'runAsNonRoot' and 'runAsUser', if you prefer, just to be extra declaritive. 


## Privileged mode

A pod's containers only has a limited level of access to worker node's kernel. To get full access, you need to enable the privileged option:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-privileged
spec:
  containers:
    - name: cntr-alpine
      image: alpine
      securityContext:
        privileged: true                  # Add this line
      command: ["sh", "-c"]
      args:
        - |
          while true ; do
            date
            id 
            whoami
            sleep 10 
          done
```

One of the main things this gives is access to all the worker node's devices, such as network devices e:

```bash
$ kubectl exec pod-alpine-default  --  ls -l /dev | wc -l
      17

$ kubectl exec pod-privileged -- ls -l /dev | wc -l
     141
```

This kind of setting is normally used by kube-system pods to make changes to the worker node, e.g. kube-proxy uses this to update the worker node's iptables. 


## Capabilities

Pods by default only has access to limited number of kernel capabilities. For example you can't change the time:

```bash
$ kubectl exec pod-alpine-default -- date +%T -s "12:00:00"
date: can't set date: Operation not permitted
12:00:00
```



We can overcome this by enabling the particular capabilities that's required. The ubuntu man pages lists these capability:

```bash
man capabilities
```

For example to give the container the ability to change it's own time, we do:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-capabilities
spec:
  containers:
    - name: cntr-alpine
      image: alpine
      securityContext:
        capabilities:                     # We add this line. 
          add:                           
            - SYS_TIME
            - NET_ADMIN
          drop:
            - CHOWN
      command: ["sh", "-c"]
      args:
        - |
          while true ; do
            date
            sleep 10 
          done
```



This results in:


```bash
$ kubectl exec pod-capabilities -- date 
Fri Apr 19 19:06:21 UTC 2019

$ kubectl exec pod-capabilities -- date +%T -s "12:00:00"
12:00:00

$ kubectl exec pod-capabilities -- date 
Fri Apr 19 12:00:02 UTC 2019
```

You can also remove capabilities, like the way we removed the ability to use the chown command:


```bash
$ kubectl exec pod-capabilities -it -- sh
/ # cd /home/
/home # touch testfile.txt
/home # ls -l
total 0
-rw-r--r--    1 root     root             0 Apr 19 19:09 testfile.txt

/home # cat /etc/passwd | grep guest
guest:x:405:100:guest:/dev/null:/sbin/nologin

/home # chown guest testfile.txt 
chown: testfile.txt: Operation not permitted
```

## readOnlyRootFilesystem

The readOnlyRootFilesystem setting, as the name implies make the container's filesystem readonly. This forces your container to make use of volumes and persistent volumes for storing data. 


```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-readonlyrootfs
spec:
  containers:
    - name: cntr-alpine
      image: alpine
      securityContext:
        readOnlyRootFilesystem: true                 # We add this line
      command: ["sh", "-c"]
      args:
        - |
          while true ; do
            date
            sleep 10 
          done
```

This results in the following behaviour:

```bash
$ kubectl exec pod-alpine-default -it -- sh
/ # touch /home/testfile.txt
/ # ls -l /home/testfile.txt 
-rw-r--r--    1 root     root             0 Apr 19 23:05 /home/testfile.txt
/ # exit

$ kubectl exec pod-readonlyrootfs -it -- sh
/ # touch /home/testfile.txt
touch: /home/testfile.txt: Read-only file system
```

This is handy setting to use if you want to avoid accidently writing data to your container's root filesystem that then ends up getting deleted, when the container gets deleted/recreated. 


## Pod level Security Contexts

This features is useful in a couple of ways. Firstly it can be used to cut down on duplicate lines. E.g. if you have multiple containers in a pod, and you want all these containers to have the same runAsUser setting, then specify it using `pod.spec.securityContext.runAsUser`.

Some settings are pod level specific security contexts. For example when 2 containers mounts the a shared volume, then the files/folders created in the shared volume need to have the same group ownership. Otherwise you can end up with situation where one pod's contaienr creates files/folders that are not accessible by another one of the pod's container. To solve this problem you can use the 'filesystem group' setting, `pod.spec.securityContext.fsGroup`. 


```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-fsgroup
spec:
  securityContext:                                   # we add this section
    fsGroup: 3000
  volumes: 
    - name: webcontent
      emptyDir: {}
  containers:
    - name: cntr-httpd
      image: httpd
      volumeMounts: 
        - name: webcontent
          mountPath: /usr/local/apache2/htdocs
      ports:
        - containerPort: 80
    - name: cntr-centos
      image: centos
      volumeMounts:
        - name: webcontent
          mountPath: /tmp/reports
      command: ["/bin/bash", "-c"]
      args:
        - |
          while true ; do
            echo "You've hit $(hostname)" >> /tmp/reports/index.html
            date >> /tmp/reports/index.html
            sleep 20
          done
```


This setting only takes effect when working with files/folders inside a mounted volume:

```bash
$ kubectl exec pod-fsgroup -it -c cntr-httpd  -- bash 
root@pod-fsgroup:/usr/local/apache2# touch /tmp/testfile.txt                 
root@pod-fsgroup:/usr/local/apache2# ls -l /tmp/testfile.txt
-rw-r--r-- 1 root root 0 Apr 20 12:58 /tmp/testfile.txt


root@pod-fsgroup:/usr/local/apache2# touch /usr/local/apache2/htdocs/testfile.txt
root@pod-fsgroup:/usr/local/apache2# ls -l /usr/local/apache2/htdocs/testfile.txt
-rw-r--r-- 1 root 3000 0 Apr 20 12:59 /usr/local/apache2/htdocs/testfile.txt
```

It shows 3000 here, rather than a group name because we don't have a group with the gid value of 3000, so the group is simply the gid value:

```bash
root@pod-fsgroup:/usr/local/apache2# cat /etc/group | grep 3000
root@pod-fsgroup:/usr/local/apache2#

root@pod-fsgroup:/usr/local/apache2# id
uid=0(root) gid=0(root) groups=0(root),3000
```






## Node level capabilities

The following makes changes at worker node level:

- `pod.spec.hostNetwork`
- `pod.spec.containers.ports.hostPort`
- `pod.spec.hostPID`
- `pod.spec.hostIPC`
- `pod.spec.containers.securityContext.privileged`

These kind of features are typically used by pods in the kube-system namespace, and causes behaviour that makes it appear that the pod's application are installed directly on the worker nodes. 

## References
[https://medium.com/devopslinks/kubernetes-pod-security-101-15fe8cda829e](https://medium.com/devopslinks/kubernetes-pod-security-101-15fe8cda829e)

[https://medium.com/coryodaniel/kubernetes-assigning-pod-security-policies-with-rbac-2ad2e847c754](https://medium.com/coryodaniel/kubernetes-assigning-pod-security-policies-with-rbac-2ad2e847c754)

[https://kubernetes.io/docs/concepts/workloads/pods/pod/#privileged-mode-for-pod-containers](https://kubernetes.io/docs/concepts/workloads/pods/pod/#privileged-mode-for-pod-containers)