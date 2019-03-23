# useful links

main exam homepage: [https://www.cncf.io/certification/cka/](https://www.cncf.io/certification/cka/)


- [KubernetesByExample](http://kubernetesbyexample.com/)

- [https://github.com/cncf/curriculum](https://github.com/cncf/curriculum)
- [https://medium.com/@ContinoHQ/the-ultimate-guide-to-passing-the-cka-exam-1ee8c0fd44cd](https://medium.com/@ContinoHQ/the-ultimate-guide-to-passing-the-cka-exam-1ee8c0fd44cd)
- [https://medium.com/talking-tech-all-around/how-to-nail-the-cka-exam-7944be5476a4](https://medium.com/talking-tech-all-around/how-to-nail-the-cka-exam-7944be5476a4k)
- [https://www.reddit.com/r/kubernetes/comments/ajcjy0/75_tips_to_help_you_ace_the_certified_kubernetes/](https://www.reddit.com/r/kubernetes/comments/ajcjy0/75_tips_to_help_you_ace_the_certified_kubernetes/)
- [https://training.linuxfoundation.org/training/kubernetes-fundamentals/](https://training.linuxfoundation.org/training/kubernetes-fundamentals/)
[https://medium.com/@pmvk/tips-to-crack-certified-kubernetes-administrator-cka-exam-c949c7a9bea1](https://medium.com/@pmvk/tips-to-crack-certified-kubernetes-administrator-cka-exam-c949c7a9bea1)
[https://medium.com/akena-blog/k8s-admin-exam-tips-22961241ba7d](https://medium.com/akena-blog/k8s-admin-exam-tips-22961241ba7d)
[https://medium.com/@krystiannowaczyk/how-i-passed-the-cka-certified-kubernetes-administrator-exam-f94b11566528](https://medium.com/@krystiannowaczyk/how-i-passed-the-cka-certified-kubernetes-administrator-exam-f94b11566528k)
[]()
[]()
[]()



Allowed websites to access during exam:

 [https://kubernetes.io/docs/](https://kubernetes.io/docs/) and its subdomains, 
 [https://github.com/kubernetes/](https://github.com/kubernetes/) and its subdomains, 
 or [https://kubernetes.io/blog/](https://kubernetes.io/blog/)


Need to purchase exam via the cncf website, to be eligible for a free retake. (see retake policy section in pdf)


 # exam tips:

1. Always start by running the following to get autocompletion working:

```bash
source <(kubectl completion bash) # see the cheatsheet page
```

2. quick create yaml file templates:

```bash

# this creates a service yaml file
kubectl expose pod podname --type=NodePort --name servicename -o yaml --dry-run
kubectl create secret generic mysql-secrets --from-literal MysqlRootPassword=password123 --dry-run -o yaml
```

3. If pods keeps dying, run the following command to see the previously failed pod's main container log:

```bash
kubectl logs podname --previous
```

4. Know where to find kubernetes component logs. 

The location to find kubernetes internal component logs, depends on how kubernetes was installed. 

If you installed kubernetes using kubeadm, then a lot kubernetes components are in the form of pods, e.g. kube-proxy:

```bash
kubectl log kube-proxy-bnzr6 --namespace=kube-system
kubectl log coredns-86c58d9df4-lnrv8 --namespace=kube-system
kubectl log etcd-kube-master --namespace=kube-system
kubectl log kube-apiserver-kube-master --namespace=kube-system
kubectl log kube-scheduler-kube-master --namespace=kube-system
kubectl log kube-controller-manager-kube-master --namespace=kube-system
```

You can exec into all these pods, and run `ps -ef | less` This will show where the configs are located. Or do `kubectl describe pods podname` to see the entrypoint command.   

A lot of these pods also sends logs to the master/worker node's /var/log/containers directory, using hostPath. 

Other components are run as systemctl services:

```bash
systemctl status docker
systemctl status kubelet
```



You can also find samples in the api documentations:

https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#service-v1-core