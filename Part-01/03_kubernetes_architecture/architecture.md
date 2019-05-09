# What is Kubernetes

If you look take a look at the official documentation, you'll find [Kubernetes is defined as](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/):

> Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

Now that's quite a mouthful, and the documentation goes into a lot more detail. Ultimately Kubernetes is a solution for managing the running of containers.

Docker can also do the same so you might wonder, why do you need Kubernetes when you have Docker? Theres a lot of reasons why, but lets take a look at one scenario:

![docker-server](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Part-01/03_kubernetes_architecture/images/Docker-Server-Architecture.png)

In this scenario we have a IT Admin who uses the docker cli to create a collection of containers on a docker server. The Docker server could a be on anything, a physical server, a VM on an on premise Data Center, a EC2 instance in AWS,..etc. These containers delivers a online e-commerce website that online shoppers can use to make purchases. Thre are a few problems with this setup:

- You can only scale vertically, i.e. if the docker server is running out of cpu and ram, then you can only increase the cpu and ram of the existing docker server.  
- No HA. If docker server crashes then all containers die with it, and it can take several minutes to build a replacement docker server.

One option is to build multiple docker servers and put them behind a Load balancer. But that's a workaround rather than a proper solution. What's needed is a container orchestration solution, such as [Docker Swarm](https://docs.docker.com/engine/swarm/), or in our case Kubernetes.


## Kubernetes Components

The Kubernetes software is made up over [several smaller self-contained inter-connecting components](https://kubernetes.io/docs/concepts/overview/components/) that are all working together to deliver a single instance of a Kubernetes Platform. These components falls in of 2 categories, Master Components and Node Components.

The master components manages the Kubernetes platform as a whole, whereas Node components are responsible for the actual running of the containers.

![kubernetes-server](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Part-01/03_kubernetes_architecture/images/kubernetes-components.png)

### master components

Here's a brief description of the master components, Don't worry if these makes sense to you at this stage, but it will as we progress through the course:

- **etcd** - this is a key/value datastore to store the current state of the kube cluster.
- [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) - This is single binary that we run using a daemon. this recieves instructions from the kubectl cli tool
- **kube-controller-manager** - Manages controller objects
- **kube-scheduler** - Determines which pods should be deployed to which worker node. 

### worker components

Here's a brief description of the worker components, Don't worry if these makes sense to you at this stage, but it will as we progress through the course:

- **containerd** - main runtime engine used for runing containers
- **kubelet** - receives instructions from master components about what pods should be running then sends instructions to containerd
- **kube-proxy** - manages networking across all worker nodes. It creates a network overlay, so that each pod has it's own unique ip address

## Kubernetes Single Node Setup

There's actually lots of different way to install kubernetes. One option is to install all the components onto a single node:

![Single-node-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Part-01/03_kubernetes_architecture/images/Single-node-kubecluster.png)

In Kubernetes, your containers run inside a Kubernetes construct called pods.

This setup suffers from the same problems as the Docker setup that we saw earlier, i.e. you can only scale vertically, and there's no HA. 

That's why a single node setup is only appropriate building a Kubernetes development environmont, because it's quick and cheap to setup. There's a tool called Minikube that specifically used create a Kubernetes instances that' in the form of a single VM that's running locally on your macbook/laptop.  

## Kubernetes Multi Node Setup

One of the main reasons Kubernetes was developed with a modular design, is so that you can install a single instance of Kubernetes across multiple nodes:

![multi-worker-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Part-01/03_kubernetes_architecture/images/multi-worker-kubecluster.png)

A collection of kube masters and workers that provides a single working instance of Kubernetes, is referred to as a **Kubernetes Cluster**, or just **Kube Cluster**.

Building a Kube Cluster involves building 2 types of servers:

- **Kube Master** - This server is responsible for managing the Kube Cluster as a whole and it has all the master components installed on it. They are also referred to as 'controller nodes'.
- **Kube worker** - These servers are repsonsible for running the actual containers (in the form of pods), that the Kube Master instructs them to run. 

This setup now allows for horizontal scaling by simply adding/removing Kube workers as and when needed. Also if one of the kube worker dies, then the kube master will become aware of it and create pods in the remaining worker nodes to compensate for it.

However this setup still has a single point of failure, which is the kube master itself. That's why in order to achieve HA you need to have multiple kube masters in your kube cluster:

![ha-kubecluster.png](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide/raw/master/Part-01/03_kubernetes_architecture/images/ha-kubecluster.png)

In order for HA to work properly, you need to have an odd number of Kube masters in your cluster, e.g. 3, 5, 7. These kube masters together are referred to as the **control plane**.


Along with those, you also have:

- local workstation - this has kubectl installed on it to let you manage your Kubernetes cluster.
- loadbalancer - used to loadbalance kube-api data traffic to the kube masters. In particular it forwards traffic to the kube-apiserver component. the traffic may originate from:
  - local workstations - when someone uses the kubectl command to perform tasks
  - The kubelet or kube-proxy components that are running inside worker nodes.