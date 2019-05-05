# Install Kubernetes on MacOS

To run a kubecluster locally on your macbook, you need to install a software called [Minikube](https://kubernetes.io/docs/setup/minikube/). This.

There are a few steps involved in this process:

1. Install [brew](https://brew.sh/) - this is a MacOS based package installer.
2. run: `brew install kubectl` to install kubectl
3. run: `brew cask install virtualbox` to install virtualbox
4. run: `brew cask install minikube` to install minikube

After minikube is installed, next check it's version:

```bash
$ minikube version
minikube version: v1.0.0
```

Similarly you check it's status:

```bash
$ minikube status
host:
kubelet:
apiserver:
kubectl:
```

To create a new kubecluster on your macbook, run (note this can take several minutes):

```bash
$ minikube start
😄  minikube v0.34.1 on darwin (amd64)
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
💿  Downloading Minikube ISO ...
 184.30 MB / 184.30 MB [============================================] 100.00% 0s
📶  "minikube" IP address is 192.168.99.100
🐳  Configuring Docker as the container runtime ...
✨  Preparing Kubernetes environment ...
💾  Downloading kubeadm v1.13.3
💾  Downloading kubelet v1.13.3
🚜  Pulling images required by Kubernetes v1.13.3 ...
🚀  Launching Kubernetes v1.13.3 using kubeadm ...
🔑  Configuring cluster permissions ...
🤔  Verifying component health .....
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
```

If you open up the virtualbox gui, you should a new vm called minikube running. If you check the status again, you should now see:

```bash
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

Here it says that minikube has also configured kubectl, to check if that's really the case, run:

```bash
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/proxy/namespaces/kube-system/services/kube-dns

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Another command can run to see the health of your kub cluster's control plane:

```bash
$ kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

Also to see how many nodes are in our kubecluster, run:

```bash
$ kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   20h   v1.14.0   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.6.2
This command lists out all VMs that has the kubelet component running on it. Also the VERSION lists the version of the kubelet. If you built kubernetes the hardway then the masters won't get listed here, since the masters don't have the kubelet running on them.

By design, to stay lightweight, our minikube based kubecluster only has one node, which acts as both the master and worker node. That's fine in a development environment. But in production, you should have multiple master and worker nodes for HA.

## Configuring the kubectl cli

When you ran, `minikube start` earlier, what actually happened to configure kubectl cli, is that the yaml file `~/.kube/config` was created (or updated). This file is referred to as a 'kubeconfig' file. The kubectl cli can only interact with one kubecluster at a time. However the `~/.kube/config` can store settings for multiple kubeclusters, and you can switch kubectl to connect to a different kube context by running:

```bash
kubectl config use-context context-name
```

A [context is basically a selection of info kubectl needs to connect to a kubecluster](https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease), we'll cover more about this later.

At the moment our `~/.kube/config` contains connection settings info for 3 contexts:

```bash
$ kubectl config get-contexts
CURRENT   NAME                 CLUSTER                      AUTHINFO             NAMESPACE
          default              kubernetes                   chowdhus
          docker-for-desktop   docker-for-desktop-cluster   docker-for-desktop
*         minikube             minikube                     minikube
```

Here we can see that kubectl is currently configured to interact with the 'minikube' cluster.

'kubeconfigs' is actually a more general terms and is used to refer to any config files used by various kubernetes components (kubelet, kube-proxy, kube-scheduler,...etc). You can [generate all these various types of kubeconfigs using the kubectl command](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/05-kubernetes-configuration-files.md#the-kubelet-kubernetes-configuration-file).

We'll cover more about the `~/.kube/config` kubeconfig file later on.

## Using Docker with Minikube

The kubecluster running inside the minikube vm actually uses Docker to run all the containers. So when you create kubernetes objects, e.g. pods, then you can use the docker cli to view the underlying containers that have been created. You might want to do this for troubleshooting/debugging purposes.

In our macbooks, the docker cli is preconfigured to interact with the docker daemon that's running directly on our macbook. At the moment our macbook isn't directly running any containers:

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

However to interact with the minikube's docker daemon, you need to configure your macbook's docker cli to connect to your minikube's docker daemon. That's done by simply setting some environment variables, DOCKER_HOST, DOCKER_CERT_PATH,...etc. The minikube cli helpfully provides these environment variables:

```bash
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/schowdhury/.minikube/certs"
export DOCKER_API_VERSION="1.35"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

This output also give you a one-liner setup:

```bash
eval $(minikube docker-env)
```

This will only make the changes for the current session and will get reset when you restart are your bash terminal (to make this change permenant you need to add this into your bash profile script files). You should now see something like:

```bash
$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes                                                                             k8s_default-http-backend_default-http-backend-5ff9d456ff-m62k8_kube-system_cee7bc7a-4001-11e9-9566-080027d15c4c_0
da459f1cfb48        quay.io/kubernetes-ingress-controller/nginx-ingress-controller   "/entrypoint.sh /ngi…"   2 minutes ago       Up 2 minutes                                                                             k8s_nginx-ingress-controller_nginx-ingress-controller-7c66d668b-xq5gj_kube-system_cf8d09f1-4001-11e9-9566-080027d15c4c_0
...
```

Notice that we already have some containers running, that's because these containers are used by kubernetes itself for it's internal workings. If you don't have docker installed on your macbook, then you can still use the docker cli that's preinstalled inside the minikube vm. You do that by first ssh'ing into the minikube VM:

```bash
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)




$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes
...
```

## The Kubernetes dashboard

You can also monitor your kubecluster via the web browser, bu running:

```bash
$ minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:50387/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

### References

[https://kubernetes.io/docs/setup/minikube/](https://kubernetes.io/docs/setup/minikube/)
[https://kubernetes.io/docs/tutorials/hello-minikube/](https://kubernetes.io/docs/tutorials/hello-minikube/)
[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/tutorials/hello-minikube/)  (talks about getting autocomplete to work)
