While using Kubernetes can be fun and intuitive, the same unfortunately can't be said about setting up a kube cluster. 

Actually that's not true, it is quite easy if you want to set up a kube cluster for testing and development purpposes. But it can get hard if you want to build a production grade cluster.  


There's lots of install options

1. The hardway - great for learning purposes
3. local workstation dev environment - minikube,...what are the otherss?
2. kubeadm - bootstrap approach. Easier approach. 
3. Third party tools - e.g. kops, -  pro: production ready clusters cons: learning curve, too many competing tools, maintenance falls on you, if cluster has problems
4. Vendor specifiy Managed services approach, e.g. aws eks. pro: turnkey approach  con: don't have full control, vendor lockin. 
5. Different flavours of Kubernetes, openshift, rancher, ...etc. 



While Kubernetes is great for simplifying your IT estate through containerisation. 

Building a Kube Cluster itself is not that straight forward. It involves having to answer lots of questions, for example:

- do you want to build a development kube cluster or production
- assuming it's production, how many masters and worker nodes do you want your cluster to be made up of. Will the worker node's specs be enough to handle all the workload
- where do you want to run your cluster, on prem or cloud
- if on cloud, which cloud? AWS, Azure,...etc. What are 
- If on cloud, Do you want to install our cluster by creating your VMs first and then bootstrapping kubernetes onto them using the kubeadm?
- Or use on the other 3rd party tools such as kops, kubespray, ...etc. 
- Or take advantage of the automated cloud offerings such as aws EKS. 
- Other kubernetes versions, e.g. rancher and openshift

As you can, there are lots of things to consider. In fact going through all of the above would take a good few hours and that's before we even start using Kubernetes. There are pro and cons to the choice you end up making. 

If 





The kubernetes is made up of several individual components that needs to be installed. 

Setting up a Kubernetes cluster is complicated. 


That's why there are lots of ways to set install and 