There's more than 1 way to skin a cat. And the same can be said about setting up a kubernetes Cluster. There are lots of ways to setup a kubernetes cluster. So let's take a look at some of the main ways. 

If you want a cluster just for learning and experimenting then minikube is a good option for that. Minikube can spin up a cluster locally on your workstation. There's also other tools that does a similar thing, such as kind. If you're workstation doesn't have enough cpu and memory to use minikube, then you can try using digital ocean instead. Digital Ocean is a cloud platform like AWS, but it's tailored for for software developers and people who are new to the cloud. It's really easy to use and you can use it for free using the following coupon - I'll cover more about Digital Ocean in the next video. 




Another great learning resource is Kubernetes the hardway. This let's you manually build your kubernetes cluster by installing all the components. This is quite an advanced guide and I would suggest you try it out at the end of this course. 


However it's definitely worth doing it help you understand how all the indviduals components fit together. 


Next we have kubeadm. This is an official kubernetes tool that effectively bootstraps a cluster with a production grade setup. We'll be demo this later in the course. 



Next we have tools developed by the wider kubernetes community. Here are few of them:




Some of these tools are aimed at particular cloudplatforms. 



Next we have kubernetes flavoured installs - 

openshift
rancher

These are essentially kubernetes with a host of advanced features. Openshift for example markets itself as an enterprise version of Kubernetes. 



Finally we have saas options. Such as AKS, EKS. 

These can be used to build production grade kube clsuters. But they have vendor lockin and less control, but easier to maintain since it's the cloud vendors update to maintain them. Such as patching the cluster with the latest security fixes, and keeping the os and kubernetes up to date. 

Digital Ocean also has a 1-click install option, and that's what we'll demo in the next video. . 



