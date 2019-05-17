# What is Kubernetes

Kubernetes is a container orchestration platform that was originally developed in-house by Google. In 2014 they open-sourced and made it available to the public. If you take a look at the official documentation, you'll find the following [definition of Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/):

*Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.*

Basically this definition means Kubernetes manages the running of containers. And these containers runs inside a construct called **Pods**. Pods are the fundamental building block in Kubernetes.

In practice that means that Kubernetes is used for things like:

- Running containers, which in Kubernetes is done by creating pods.
- Running a group identical pods across multiple servers
- Load balance traffic across these pods
- Deploying newer pods to replace existing pods
- Setting up networking so that pods can reach one another

Don't worry if none of these makes any sense. We'll be covering them as we go through the course. 

## Kubernetes Components

Kubernetes is not a single piece of software, in fact it's made up of [several components](https://kubernetes.io/docs/concepts/overview/components/) that work together to provide a single instance of Kubernetes. These components are categorised into 2 groups:

- **Master Components** - These manages the Kubernetes platform as a whole
- **Node Components** - These are responsible for the actual running of pods.

![kubernetes-server](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/kubernetes-components.png)

### master components

Here's a brief description of the master components, Don't worry if these makes sense to you at this stage, but it will as we progress through the course:

- **etcd** - this is a key/value datastore to store the current state of the kube cluster.
- **[kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)** - This is single binary that we run using a daemon. this recieves instructions from the kubectl cli tool
- **kube-controller-manager** - Manages controller objects
- **kube-scheduler** - Determines which pods should be deployed to which worker node. 

### worker components

Here's a brief description of the worker components, Don't worry if these makes sense to you at this stage, but it will as we progress through the course:

- **Container Runtime Engine** - This component is responsible for the actual running of containers. There are a few options available, such as Containerd and Docker.  
- **kubelet** - receives instructions from master components about what pods should be running then sends instructions to containerd
- **kube-proxy** - manages networking across all worker nodes. It creates a network overlay, so that each pod has it's own unique ip address


## Kubernetes Single Node Setup

There's actually lots of different ways to setup kubernetes. One option is to install all the components onto a single node:

![Single-node-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/Single-node-kubecluster.png)

This setup suffers from the same problems as the Docker setup that we saw earlier, i.e. you can only scale vertically, and there's no HA.

That's why a single node setup is only appropriate when building Kubernetes development environmonts, since it's quick and cheap to setup. For example, Minikube is used for building a single-VM Kubernetes instance that runs locally on your workstation.  

## Kubernetes Multi Node Setup

One of the main reasons Kubernetes was developed with a modular design, is so that you can install a single instance of Kubernetes across multiple nodes:

![multi-worker-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/multi-worker-kubecluster.png)

A collection of kube masters and workers that provides a single working instance of Kubernetes, is referred to as a **Kubernetes Cluster**, or just **Kube Cluster**.

Building a Kube Cluster involves building 2 types of servers:

- **Kube Master** - This server is responsible for managing the Kube Cluster as a whole and it has all the master components installed on it. They are also referred to as 'controller nodes'.
- **Kube worker** - These servers are repsonsible for running the actual containers (in the form of pods), that the Kube Master instructs them to run. 

This setup now allows for horizontal scaling by simply adding/removing Kube workers as and when needed. Also you have HA, because if one of the kube workers die, then the kube master will become aware of it and create new pods in the remaining worker nodes to replace the lost ones.

However this setup still has a single point of failure, which is the kube master itself. That's why in order to achieve HA you need to have multiple kube masters in your kube cluster:

![ha-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/ha-kubecluster.png)

In order for HA to work properly, you need to have an odd number of Kube masters in your cluster, e.g. 3, 5, 7. These kube masters together are referred to as the **control plane**.

Along with those, you also have:

- local workstation - this has kubectl installed on it to let you manage your Kubernetes cluster.
- loadbalancer - used to loadbalance kube-api data traffic to the kube masters. In particular it forwards traffic to the kube-apiserver component. the traffic may originate from:
  - local workstations - when someone uses the kubectl command to perform tasks
  - The kubelet or kube-proxy components that are running inside worker nodes.