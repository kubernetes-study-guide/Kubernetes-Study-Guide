# architecture

Let's say you want to run all your containers ona single docker server. Then you have two problems:

- The more containers you run the more powerful vm you need. So will have to do vertical scaling. 
- No HA. If docker server crashes then all containers die. 

That's why a better approach is to run all your containers using kubernetes. There's actually lots of different way to install kubernetes. One of the simplest ways is to use minikube. 


However minikube presents the same problem. 







The Kubernetes software is not a single software, it is in fact a highly modular software that's made up of several independent components. Some of these components controls the actual internal runnings of kubernetes, whereas other components are responsible for running the actual containers. 

. You can install all these components on a single vm. Which is the case of minikube. 


However for Horizontal scaling and HA, it's best to run the ac


You can install all these components on a single machine, which is the case with minikube. But for better redundancies, HA, scalability it is better to install the components a cluster of vms. 



A production ready kuberenetes setup is made up of several servers. they are either:

- master nodes, aka controller nodes, control plane
- worker nodes - runs pods


Along with those, you also have:

- local workstation - this has kubectl installed on it. 
- loadbalancer - used to loadbalance kube-api data traffic to master nodes. In particular is forwards traffic to the kube-apiserver component. the traffic may originate from:
  - local workstations - when someone uses the kubectl command to perform tasks
  - The kubelet or kube-proxy components that are running inside worker nodes. 



## master nodes

Master nodes have the following components installed on them:

- etcd - this is a key/value datastore to store the current state of the kube cluster.
- [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) - This is single binary that we run using a daemon. this recieves instructions from the kubectl cli tool
- kube-controller-manager
- kube-scheduler


## worker nodes

- containerd - main runtime engine used for runnng containers
- kubelet - receives instructions from master nodes about what pods should be running then sends instructions to containerd
- kube-proxy - manages networking across all worker nodes. It does:
  - dns 
  - Creates network overlay, so that each pod has it's own unique ip address