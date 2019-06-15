# Transcript

slides only




Hello Everyone and Welcome back.  

So what is Kubernetes?

Kubernetes is a container orchestration platform,  that was originally developed in-house by Google. 

In 2014 they open-sourced it and made it available to the public. 

In the official documentation, it is defined as follows:

*Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, 
that facilitates both declarative configuration and automation.*

Basically this definition means Kubernetes manages the running of containers. 
In Kubernetes you're not allowed to have containers running on their own. 
Instead containers run inside a construct called **Pods**. You can have 
multiple containers running inside a single pod. This makes Pods the fundamental building 
block in Kubernetes.

In practice, Kubernetes is used for things like:

- Creating and running pods.
- Running a group identical pods across multiple nodes
- Load balancing traffic across these pods
- Deploying newer pods to replace existing pods
- Setting up networking so that pods can talk to one another

Don't worry if none of these makes any sense. We'll be covering them as we go through the course. 


Kubernetes is a platform that's made up of several smaller components that work together to 
provide a single working instance of Kubernetes. These components fall into 2 groups:

- **Master Components** - which manages the Kubernetes platform as a whole
- **Worker Components** - which are responsible for the actual running of pods.

![kubernetes-server](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/kubernetes-components.png)

All these components are installed on one or more "nodes". In Kubernetes, a node is an 
instance of a Linux OS. It doesn't matter what the node's underlying hardware is, 
it can be a VM running locally on your workstation, or a VM in the cloud, 
or a physical server, or even a raspberry pi! 


So what are the master components? they are:

- etcd - this is a key/value datastore to store Kubernetes's current state. It's a bit like Kubernetes's own internal database.
- kube-apiserver - This is the front facing part of kubernetes. It is component that kubectl interacts with
- kube-controller-manager - This  Manages the controller objects
- kube-scheduler - This Determines which pods should be deployed to which worker node. 

Again, don't worry if this doesnt make much sense to you. We'll cover them as we go through the course. 


So what are the worker components? they are:


- Container Runtime Engine - This component is responsible for the 
  actual running of the containers. 
  There are a few Container Runtime Engines to choose from, such as Containerd and Docker. 
  
  Next we have the

- kubelet - This receives instructions from master components about what pods should be 
  running then sends instructions to the Container Runtime Engine to build the required containers. 
  
  and finally we have the 

- kube-proxy - This manages networking across all worker nodes. It creates a network overlay, 
  so that each pod has it's own unique ip address





There's actually lots of different ways to setup kubernetes. One option is to install all the components onto a single node:

![Single-node-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/Single-node-kubecluster.png)

This setup isn't recommended for production environments because

- you are limited to vertical scaling - Meaning that you can only increase capacity 
  by adding more cpu and ram to the existing node. 
- There's no High Availability - So if the node dies then there will be some 
  downtime until a replacement node is built. 

That's why a single node setup is only appropriate when building development environmonts, 
since it's quick and easy to setup. 
Minikube is commonly used for building this type of single-vm setup.  

It's actually recommended to install a single instance of Kubernetes across multiple nodes:

![multi-worker-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/multi-worker-kubecluster.png)

A collection of kube masters and workers that provides a single working instance of Kubernetes, is referred to as a **Kubernetes Cluster**, or just **Kube Cluster**.

Kube Cluster are made up of 2 types of nodes:

- Kube Masters - This server is responsible for managing the Kube Cluster as a whole. It has all the master components installed on it. They are also referred to as 'controller nodes'.
- Kube Workers - These servers are responsible for running the actual pods, that the Kube Master instructs them to run. They have all the worker components installed on them.  

This setup now allows for horizontal scaling by simply adding or removing Kube workers as and when needed. Also you have HA, in the sense that if one of the kube worker nodes dies, then the kube master will become aware of it and create new pods in the remaining worker nodes to make up for it.

However this setup still has a single point of failure, which is the kube master itself. That's why in order to achieve HA you need to have multiple kube masters in your kube cluster:

![ha-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Section-01/04_What_is_Kubernetes/images/ha-kubecluster.png)

In order for HA to work properly, you need to have an odd number of Kube masters in your cluster, e.g. 3, 5, 7. These kube masters together are referred to as the **control plane**.

##TODO-START
Finally I should mention that Kubernetes doesn't come with feature to allow pod-to-pod across worker nodes. That feature is something that's added into a kube cluster by installing third party networking plugins. These networking plugins provide the following features:
- all containers can communicate with eachother without resorting to NAT. 
- all nodes can communicate with all containers without resorting to NAT. 

These plugins are referred to as cni plugins. Where cni stands for contanier networking interface.
##TODO-END
That's it for this video, see you in the next one. 