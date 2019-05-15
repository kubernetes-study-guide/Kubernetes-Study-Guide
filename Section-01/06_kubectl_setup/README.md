# Configuring the kubectl cli

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
