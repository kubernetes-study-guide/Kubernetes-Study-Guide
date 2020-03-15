# kube-api-server


This is the only component that interacts with etcd. 

The options used for starting the kube-apiserver are stored in either as 

- kube-apiserver.service file, if installed the hardway
- inside the kube-apiserver pod if installed as a pod by a tool such as kubeadm. 