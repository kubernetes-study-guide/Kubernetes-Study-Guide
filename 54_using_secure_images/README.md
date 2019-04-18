# Using secure images

Up until now, all the images we used were public images that we pulled down from docker hub. We didn't need to configure docker hub, since kubernetes uses docker hub as the default repo. 

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
$ kubectl create secret docker-registry docker-hub-credentials --docker-server=https://index.docker.io/v1/ --docker-username=codingbee --docker-password=hotelviewdenmark --docker-email=not-used@ignore.com
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


Now lets create a pod that makes use of this secret to pull down a private repo:


```bash


```





## References
[https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry](https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry)

[https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
