# Install Kubernetes using Minikube

There are lots of other ways to build a kube cluster, such as [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/), or my favourite, [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way). However, we will create a kube cluster locally on our workstation using [Minikube](https://kubernetes.io/docs/setup/minikube/). I should point that all the things I'll demo in this course will work the same way irrespective of how the Kube Cluster was built.

Earlier we talked about how to install minikube, lets now if it has really installed by running a minikube command:

```bash
$ minikube version
minikube version: v1.0.1
```

Similarly you check it's status:

```bash
$ minikube status
host:
kubelet:
apiserver:
kubectl:
```

To create a new kubecluster,  we run (note this can take several minutes):

```bash
$ minikube start
😄  minikube v1.0.0 on darwin (amd64)
🤹  Downloading Kubernetes v1.14.0 images in the background ...
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
📶  "minikube" IP address is 192.168.99.114
🐳  Configuring Docker as the container runtime ...
🐳  Version of container runtime is 18.06.2-ce
⌛  Waiting for image downloads to complete ...
✨  Preparing Kubernetes environment ...
🚜  Pulling images required by Kubernetes v1.14.0 ...
🚀  Launching Kubernetes v1.14.0 using kubeadm ... 
⌛  Waiting for pods: apiserver proxy etcd scheduler controller dns
🔑  Configuring cluster permissions ...
🤔  Verifying component health .....
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
```

If you open up the virtualbox gui, you should see a new vm called minikube running. If you check the status again, you should now see:

```bash
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

Here it says that minikube has also configured kubectl, that's done by making changes to kubectl's config file. By default that file is located at `~/.kube/config`. We'll cover more about this file later in the course. But for now we'll confirm that this config file is currently configured to point to minikube's kube cluster:

```bash
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/proxy/namespaces/kube-system/services/kube-dns

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

The ip address shown here is the Minikube VM's ip address, which should match:

```bash
$ minikube ip
192.168.99.100
```

To check the health of your kub cluster's control plane, you can run:

```bash
$ kubectl get componentstatus
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

Also to see how many nodes are in our kubecluster, run:

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4d10h   v1.14.1
```

This command lists out all VMs that has the kubelet component running on it, along with the kubelet's VERSION. If you built kubernetes the hardway then the masters won't get listed here, since the masters don't have the kubelet running on them. We can specify the 'wide' output setting to display a little some more info: 

```bash
$ kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   20h   v1.14.0   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.6.2
```

Now that we have a kubecluster in place, we can now run the kubectl version command but this time without the --client filter flag:

```bash
$ kubectl version --short
Client Version: v1.14.1
Server Version: v1.14.1
```


By design, to stay lightweight, our minikube based kubecluster is a single node cluster, which acts as both the master and worker node. That's fine in a development environment. But in production, you should have multiple node cluster for High Availability, better performance, more CPU+RAM capacity,..etc.

When your not using minikube, you can shut it down:

```bash
minikube stop
```

You can also delete your minikube vm:

```bash
minikube delete
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

This is a really cool tool that let's you view and manage your Kubecluster visually. I encourage you to explore this tool as we progress through the course. 

### References

[https://kubernetes.io/docs/setup/minikube/](https://kubernetes.io/docs/setup/minikube/)
[https://kubernetes.io/docs/tutorials/hello-minikube/](https://kubernetes.io/docs/tutorials/hello-minikube/)
[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/tutorials/hello-minikube/)  (talks about getting autocomplete to work)
