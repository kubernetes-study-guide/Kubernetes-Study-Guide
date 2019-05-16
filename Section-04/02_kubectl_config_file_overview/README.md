# Configuring the kubectl cli

In order for kubectl to interact with a Kubecluster, it needs to know where the kubecluster is located (i.e. ip/url address) and it also needs to authenticate with it, which is commonly done using tls certificates. Both sets of info is stored in the kubectl's config file, `~/.kube/config`. This config file is written in yaml format, here's a sample:

```yaml
apiVersion: v1
kind: Config
preferences: {}
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.118:8443
  name: minikube
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
users:
- name: admin-user
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
- name: minikube
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
contexts:
- context:
    cluster: minikube
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
```

This file is structured alphabetically which makes it a little hard to make sense. So here's another version of the same file, but this time I've cosmeticly tweaked it to make it a bit easier to read:

```yaml
##
## Header
##
apiVersion: v1
kind: Config
preferences: {}

##
## List of Clusters
##
clusters:
- name: minikube
  cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.118:8443
- name: aws-dev
  cluster:
    certificate-authority: /Users/schowdhury/.kubectl/dev-aws-ca.crt
    server: https://82.3.184.245:8443
- name: azure-dev
  cluster:
    certificate-authority: /Users/schowdhury/.kubectl/dev-azure-ca.crt
    server: https://85.33.20.100:8443

##
## List of Users
##
users:
- name: dev-user
  user:
    client-certificate: /Users/schowdhury/.kubectl/certs/dev/client.crt
    client-key: /Users/schowdhury/.kubectl/certs/dev/client.key
- name: minikube
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key

##
## List of Contexts
##
contexts:
- name: minikube
  context:
    cluster: minikube
    user: minikube
    namespace: default

##
## The currently acive context
##
current-context: minikube
```

This file is made up of 5 parts. 


When I ran `minikube start`, minikube updated this config file so that kubectl connects to minikube's kube cluster. 



So to tell kubectl which user+cluster+namespace combo to use, you just select the context to use. If a context doesn't exist for a particular user+cluster+namespace then you just create a new context for that combo. 


You can edit any part of this config file by running 





Contexts let you mix and match clusters with credentials. So when you set the current-context you're effectively telling kubectl which username+credential combo to use. 



When you ran, `minikube start` earlier, what actually happened to configure kubectl cli, is that the yaml file `~/.kube/config` was created (or updated). This file is referred to as a 'kubeconfig' file. The kubectl cli can only interact with one kubecluster at a time. And the cluster kubectl is currently using is specified in the kubectl context. A [context is basically a selection of info kubectl needs to connect to a kubecluster](https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease). At the moment our `~/.kube/config` contains connection settings info for 3 contexts:

```bash
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          default              kubernetes                   chowdhus
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
*         minikube             minikube                     minikube
```

Here we can see that kubectl is currently configured to interact with the 'minikube' cluster.
You can switch kubectl to connect to a different kube context by running:

```bash
kubectl config use-context context-name
```

**kubeconfigs** is actually a more general terms and is used to refer to any config files used by various kubernetes components (kubelet, kube-proxy, kube-scheduler,...etc). You can [generate all these various types of kubeconfigs using the kubectl command](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/05-kubernetes-configuration-files.md#the-kubelet-kubernetes-configuration-file).

We'll cover more about the `~/.kube/config` kubeconfig file later on.


# Reference

[https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

[https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease](https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease)