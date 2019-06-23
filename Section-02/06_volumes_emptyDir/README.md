# emptyDir Volumes

Structure:
slides
-> vscode

# Slide

Hello everyone, and welcome back.

When a container dies, all the data inside that container get's deleted. However you can make some of your container's data persistant by making use of Kubernetes volumes. 

Kubernetes volumes are similar to other virtual block storage devices, such as docker volumes or logical volumes that are used in Linux. To use a kubernetes volume, you need to mount the volume to a folder inside your container. After that, any data that the container stores in that folder is then safeguarded from automatic deletion when the container dies. That's because behind the scene, kubernetes volumes ends up storing the data outside of the container. 

As a result when a container dies, the pod then builds a replacement container, and this new container can then carry on using the data left behind by the previous container.

There are different types of volumes, and in this video I'll demo how to setup a [emptyDir volume](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir). 


## vscode

For this demo I've opened up a bash terminal inside this video's topic folder. 

Volumes are defined as part of the pod's definition. Here's the yaml file we'll use for this demo.


```bash
tree configs/
code configs/httpd-emptydir.yaml
```

Volumes are defined using the pod.spec.volumes settings. Notice that volumes is plural, meaning that you can define a list of volumes here. But we'll just define one. So in this example I'm saying here that I want a single emptyDir volume, and I want to call it my-data. 

# TODO - Start
EmptyDir volume types do have lower level settings, but they are only optional. 

```bash
kubectl explain pod.spec.volumes.emptyDir
```

That's why i added an empty curly brace here to tell kubernetes, to use the default values for the lower level EmptyDir settings. However other volumes types, such as nfs, do have mandatory fields:

```bash
$ kubectl explain pod.spec.volumes.nfs
```
# TODO - end


Next, we have a volumeMounts section, which forms part of our container's definition. Here I'm saying I want to mount the my-data volume to the /em-en-tee/reports folder. That means that these 2 lines need to match. By the way, the mount point folder will get created automatically, if it doesn't already exist.

I also added an echo command as part of the startup shellscript, just so that it writes some dummy data to our volume. Let's now go ahead and create this pod.

```bash
$ kubectl apply -f configs/pod-emptyDir.yml
pod/pod-empty-dir-demo created
$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
pod-empty-dir-demo   1/1     Running   0          13s
```

After that's done, let's now   check that our volume has been mounted. We'll do that using the mount command and grepping for our mountpoint folder's   name:

```bash
$ kubectl exec -it pod-empty-dir-demo -- mount | grep reports
/dev/sda1 on /mnt/reports type ext4 (rw,relatime,data=ordered)
```

Here we can see that our volume is mounted and is showing up as dev sda1. Let's now check that our test     file exists inside the volume:

```bash
$ kubectl exec -it pod-empty-dir-demo -- cat /mnt/reports/data.txt
Fri Jun 21 20:03:11 UTC 2019: new container created
```

This line was added by the startup shell script, just before the inifinite while loop started. Lets write      a bit more content to this file. 

```bash
$ kubectl exec -it pod-empty-dir-demo -- vi /mnt/reports/data.txt
$ kubectl exec -it pod-empty-dir-demo -- cat /mnt/reports/data.txt
Fri Jun 21 20:03:11 UTC 2019: new container created
blah
blah
blah
```

Ok we've added a few more lines. Now let's test our volume's persistancy, by killing the container. 


```bash
kubectl get pods -o wide
$ kubectl exec -it pod-empty-dir-demo -- kill 1
kubectl get pods -o wide
```

Hmm, ok it looks like that didn't work. I was expecting the restart number to go up by one. I think what's happened here is that the kill command only killed the currently running sleep process rather than the whole shell script. As a result the infinite while loop just started up another sleep command. Nevermind, I'll just kill the container using docker instead. 

```bash
$ minikube docker-env
$ eval $(minikube docker-env)
$ docker container ls | grep cntr-centos   (copy and paste the containers name using cursor)
```

Here I tracked down the container by grepping for the container's name. Now let's kill it. 

```bash
$ docker container stop  (copy and paste)
$ kubecetl get pods -o wide
```

Now that lo oks more like it, the restart number has gone up by 1. Just to be sure, I'll also take a look at the kubernetes events logs: 

```bash
kubectl get events
```

Here we can see that our container has definitely died and got rebuilt again. Let's now check if the   data has persisted:

```bash
$ kubectl exec -it pod-empty-dir-demo -- cat /mnt/reports/data.txt
```

Ok everything looks good so far, here we can see the timestamp entry left behind by   the previous container, and the new timestamp appended by the new container. 

There's another thing you need to know about emptyDir volumes, that is that these volumes lives outside the container, but inside the pod. This has a couple of consequences. First it     means that only containers inside that pod can access this emptyDir volume. It also means emptyDir volumes are actually only partially persistent, since they will get deleted when the pod dies. Let's demo this right now by rebuilding the pod:

```bash
$ kubectl delete -f configs/pod-emptyDir.yml
pod "pod-empty-dir-demo" deleted
$ kubectl apply -f configs/pod-emptyDir.yml
pod/pod-empty-dir-demo created
$ kubectl get pods -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-empty-dir-demo   1/1     Running   0          9s    172.17.0.10   minikube   <none>           <none>
$ kubectl exec -it pod-empty-dir-demo -- cat /mnt/reports/data.txt
Sat Jun 22 10:00:20 UTC 2019: new container created
$ 
```

Here we can see the all the previous content have now disappeared. That's why emptyDir volumes are normally used for storing data that's only   of interest while the pod is alive.

In fact, the emptyDir volume type is a special case, it's the only volume type that live       inside pods.

```bash
$ kubectl explain pods.spec.volumes | less
```

The other volumes types such as AWSElasticBlockstore, are fully persistant, because they exist outside of pods.


So if you want to preserve your  data for longer, then you need make use one of these other volume types. We'll cover some of them later in the course. 

That's it for this video, see you in the next one. 