# architecture


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
- kube-apiserver - this recieves instructions from the kubectl cli tool
- kube-controller-manager
- kube-scheduler


## worker nodes

- containerd - main runtime engine used for runnng containers
- kubelet - receives instructions from master nodes about what pods should be running then sends instructions to containerd
- kube-proxy - manages networking across all worker nodes. It does:
  - dns 
  - Creates network overlay, so that each pod has it's own unique ip address