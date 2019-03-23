# ConfigMaps

When using images from docker hub, you'll often find yourself wanting to add additional configs (or override the default configs embeded inside the image) during a container launch time. Also some images needs requires customisation configuration as part of launch time, e.g. the mysql image that we saw earlier. where we used 'environment variables' (and 'secrets' when dealing with sensitive data) to feed that data during launch time.

Using Env Vars is quite crude, and doesn't scale well. That's why you should avoid overusing it. Another approach for adding your configs into a container is to use official docker hub images to create intermediary images with your custom configs baked in. This again isn't that elegant and doesn't scale very well.


To do more sophisticated container build time config changes, it's recommended to use ConfigMaps. 

ConfigMaps are Kubernetes objects for storing dictionary data, i.e. key value pairs. They value of a key/value pair can also be used to store entire config files. However you shouldn't use configmaps for storing secrets.

First we'll take a look at how to create configmaps and then we'll look at how to use them. 




##  Create configmaps (eg1-env-vars)

There's a few ways to configmap objects, but we'll create via kube oject yaml file approach.


```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: fav-fruit
data:
  fruit.name: banana
  fruit.color: yellow
```

After you have applied this, you can inspect it as follows:

```bash
$ kubectl get configmaps
NAME        DATA   AGE
fav-fruit   2      10s
$ kubectl describe configmaps fav-fruit
Name:         fav-fruit
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","data":{"fruit.color":"yellow","fruit.name":"banana"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"fav-fruit...

Data
====
fruit.color:
----
yellow
fruit.name:
----
banana
Events:  <none>
```


## Feeding configMap data into containers as Environment Variables (eg1-env-vars)


This approach is a good option for docker hub images that exposes customisation options via the setting of Environment Variables. Here how to inject env vars via the yaml:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
  labels:
    component: apache_webserver
spec: 
  containers:
    - name: cntr-centos
      image: centos
      env: 
        - name: FruitName    # This sets the name of the environment variable inside the container. 
          valueFrom:
            configMapKeyRef:
              name: fav-fruit  # this sets the name of the configmap to read from. 
              key: fruit.name  # this picks out the particular key value from the chosen configmap. 
        - name: FruitColor
          valueFrom:
            configMapKeyRef:
              name: fav-fruit
              key: fruit.color
      command: ["/bin/bash", "-c"]
      args:
        - while true ; do
            date ;
            curl -s http://svc-clusterip-httpd.default.svc.cluster.local:4000 ;
            sleep 10 ;
          done
```

This approach also has the added benefit of separating out data from the yaml files. 


To check if that has worked, we do:

```bash
$ kubectl exec pod-centos -c cntr-centos -it /bin/bash
[root@pod-centos /]# env | grep Fruit
FruitName=banana
FruitColor=yellow
```

## Upload files into containers using ConfigMaps (eg2-volumes)

Configmaps can be used to upload files into your container with the help of non-persistant volumes. These can be anything including. Let's say in our container, we want to create a new folder, /scripts, and in that folder you want to create to drop in a shell script, then first we create the following configmap to house the shell script: 

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: customscripts
data:
  healthcheck.sh: |
    #!/bin/bash
    sleep 20  # this is to allow the webserver pod to start up fully
    while true
    do
      date >> /tmp/script.log
      curl http://svc-clusterip-httpd.default.svc.cluster.local:4000 >> /tmp/script.log
      sleep 10
    done
  helloworld.sh: |
    #!/bin/bash
    echo hello world
```

Next we feed them into our pod using the following approach:


```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
spec: 
  volumes:
    - name: shellscripts
      configMap:              # this is special volume which stores content of configmaps.
        name: customscripts
        defaultMode: 0744     # this is to make shell scrips executable
  containers:
    - name: cntr-centos
      image: centos
      volumeMounts:
        - name: shellscripts
          mountPath: /scripts       # this ends up creating the folder. 
      command: ["/scripts/healthcheck.sh"]  # here, we're running one of the scripts that we injected in. 
```

Now let's check this:


```bash
$ kubectl exec pod-centos -c cntr-centos -it /bin/bash
[root@pod-centos /]# cd /scripts/

[root@pod-centos scripts]# ls
healthcheck.sh  helloworld.sh

[root@pod-centos scripts]# ps -ef 
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 13:31 ?        00:00:00 /bin/bash /scripts/healthcheck.sh
root       180     0  0 13:38 pts/0    00:00:00 /bin/bash
root       209     1  0 13:38 ?        00:00:00 sleep 10
root       210   180  0 13:38 pts/0    00:00:00 ps -ef

[root@pod-centos scripts]# cat /tmp/script.log 
Thu Mar  7 13:31:41 UTC 2019
<html><body><h1>It works!</h1></body></html>
Thu Mar  7 13:31:51 UTC 2019
<html><body><h1>It works!</h1></body></html>
.
.
...etc
```

# Targeted configmaps (eg3-subpaths)

In the above example we used /scripts as our mountpount. If we wanted to upload the shell scripts into an existing folder, e.g. /etc, then you would need to set the mountpoint to /etc. However, that wouldn't working becuase, any existing files in /etc would become inaccessible due to the mounting process. So let's walkthrough a slightly modified example:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
spec: 
  volumes:
    - name: healthscript
      configMap:
        name: customscripts
        items:                      # this bit is optional in this scenario
          - key: healthcheck.sh     # here we're making only one key/value aviable inside the volume
            path: healthcheck.sh
        defaultMode: 0744
  containers:
    - name: cntr-centos
      image: centos
      volumeMounts:
        - name: healthscript
          mountPath: /etc/healthcheck.sh
          subPath: healthcheck.sh          # Here se specify a part of the volume 
      command: ["/etc/healthcheck.sh"]
```

Now let's check that has worked:


```bash
$ kubectl exec pod-centos -c cntr-centos -it /bin/bash
[root@pod-centos /]# ll /etc/healthcheck.sh 
-rwxr--r-- 1 root root 218 Mar  7 14:42 /etc/healthcheck.sh
[root@pod-centos /]# cat /etc/healthcheck.sh 
#!/bin/bash
sleep 20  # this is to allow the webserver pod to start up fully
while true
do
  date >> /tmp/script.log
  curl http://svc-clusterip-httpd.default.svc.cluster.local:4000 >> /tmp/script.log
  sleep 10
done
[root@pod-centos /]# cat /tmp/script.log 
Thu Mar  7 14:42:23 UTC 2019
<html><body><h1>It works!</h1></body></html>
Thu Mar  7 14:42:33 UTC 2019
<html><body><h1>It works!</h1></body></html>
Thu Mar  7 14:42:43 UTC 2019
<html><body><h1>It works!</h1></body></html>

```