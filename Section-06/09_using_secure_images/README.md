# Using secure images


## The image setting
Up until now, all the images we used were public images from official [Docker hub](https://hub.docker.com/) image registry, since kubernetes uses docker hub as the default registry, if we don't specify the image's full name. But if you did explicitly specify an images full name, it would look like this:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
spec:
  containers:
    - name: cntr-httpd
      image: docker.io/codingbee/httpd:0.1      # here we used an image's fqdn
      ports:
        - containerPort: 80
```


The images full name is made up of 4 parts:

> {registry url}/{image's owner name}/{image's name}:{tag}

The 'resistry url' defaults to 'docker.io' if it's not specified. Also the tag defaults to 'Latest'. If you're using one of the [official images](https://docs.docker.com/docker-hub/official_images/), then you can omit the image's owner name, just spefify the image name. At the moment there are about [150 official images](https://hub.docker.com/search?q=&type=image&image_filter=official).


## Using private images

However it's likely that you'll want to build your own private images, and host them on a private registry. Setting up and authenticating with private registries varies from one cloud platform to the next. So in our example, we'll keep it simple by still using docker hub, but this time we'll pull down private images. 

First we need to provide kubernetes with our docker hub login credenetials. Kubernetes has a secrets object sub-type specifically for this purpose:

```bash
$ kubectl create secret --help
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

So here I'm creating a secret:


```bash
$ kubectl create secret docker-registry docker-hub-credentials --docker-server=https://index.docker.io/v1/ --docker-username=codingbee --docker-password=xxxxxx --docker-email=not-used@ignore.com
secret/docker-hub-credentials created


$ kubectl get secrets
NAME                     TYPE                                  DATA   AGE
default-token-95psk      kubernetes.io/service-account-token   3      41h
docker-hub-credentials   kubernetes.io/dockerconfigjson        1      16s


$ kubectl describe secrets docker-hub-credentials
Name:         docker-hub-credentials
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  172 bytes
```

At the moment I don't have a private image.

You can also view your secret's content by running:

```bash
kubectl get secrets docker-hub-credentials -o=jsonpath='{.data.\.dockerconfigjson}' | base64 --decode
{"auths":{"ttps://index.docker.io/":{"username":"codingbee","password":"xxxx","auth":"xxxxxx"}}}
```




Normally building a new image involves writing Dockerfiles, which is a course in it's own write. So we'll cheat and instead create a new private image by simply cloning the official httpd image as a starting point:

```bash
# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

# docker pull httpd
Using default tag: latest
latest: Pulling from library/httpd
27833a3ba0a5: Pull complete 
7df2f4a2bf95: Pull complete 
bbda6f884d14: Pull complete 
4d3dcf503f89: Pull complete 
b2f11da8a23e: Pull complete 
Digest: sha256:b4096b744d92d1825a36b3ace61ef4caa2ba57d0307b985cace4621139c285f7
Status: Downloaded newer image for httpd:latest

# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              d4a07e6ce470        2 weeks ago         132MB

# docker tag httpd codingbee/httpd:0.1$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
codingbee/httpd     0.1                 d4a07e6ce470        2 weeks ago         132MB
httpd               latest              d4a07e6ce470        2 weeks ago         132MB

# docker push codingbee/httpd:0.1
The push refers to repository [docker.io/codingbee/httpd]
3109d31e7c8d: Mounted from library/httpd 
c9591a4dbc31: Mounted from library/httpd 
9b4799ea4c4c: Mounted from library/httpd 
2bca991cdc4d: Mounted from library/httpd 
5dacd731af1b: Mounted from library/httpd 
0.1: digest: sha256:4ac9ce655eb83ea44e08cb9446564a8a8e2f05fe02d7c69ac4b15f22db4b1bcf size: 1367
```




Now lets see what happens when our pod descriptor tries to pull this image without using encryption:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
spec:
  containers:
    - name: cntr-httpd
      image: index.docker.io/v1/codingbee/httpd:0.1      # here we used an image's fqdn
      imagePullPolicy: always                       # this is to avoids using cached images
      ports:
        - containerPort: 80
```

We usually omit 'index.docker.io/v1/' part in the image name since that's what kubernetes uses by default. However we're explicit this time to show what you would need to do if you decided to use another docker registry, [quay](https://quay.io/search), AWC ECR,...etc. This ends up failing, because it tries to pull the image like a public image:

```bash
$ kubectl get pods -o wide
NAME        READY   STATUS             RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
pod-httpd   0/1     ImagePullBackOff   0          2m51s   192.168.1.71   kube-worker1   <none>           <none>

$ kubectl describe pods pod-httpd
...
Events:
  Type     Reason     Age               From                   Message
  ----     ------     ----              ----                   -------
  Normal   Scheduled  20s               default-scheduler      Successfully assigned default/pod-httpd to kube-worker1
  Normal   BackOff    18s               kubelet, kube-worker1  Back-off pulling image "index.docker.io/v1/codingbee/httpd:0.1"
  Warning  Failed     18s               kubelet, kube-worker1  Error: ImagePullBackOff
  Normal   Pulling    6s (x2 over 20s)  kubelet, kube-worker1  Pulling image "index.docker.io/v1/codingbee/httpd:0.1"
  Warning  Failed     4s (x2 over 19s)  kubelet, kube-worker1  Failed to pull image "index.docker.io/v1/codingbee/httpd:0.1": rpc error: code = Unknown desc = Error response from daemon: pull access denied for v1/codingbee/httpd, repository does not exist or may require 'docker login'
  Warning  Failed     4s (x2 over 19s)  kubelet, kube-worker1  Error: ErrImagePull

```

We also used `pod.spec.containers.imagePullPolicy` that's because kubernetes by default would try to use cached images. The index.docker.io/v1/codingbee/httpd:0.1 was originally a public image, before I made it private. Back when it was public I did pull make use of it in my kube cluster, and consequently kubernetes cached that image for future use. Having this set to 'always' does mean a slight performance cost though. 

Now let's try again, but this time using our login credentials:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
spec:
  imagePullSecrets:                               # we add this section
    - name: docker-hub-credentials
  containers:
    - name: cntr-httpd
      image: index.docker.io/codingbee/httpd:0.1   # note, we omit 'v1', I think because authentication is not done in v1
      imagePullPolicy: Always
      ports:
        - containerPort: 80
```

This time it is successful.








## References
[https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry](https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry)

[https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
