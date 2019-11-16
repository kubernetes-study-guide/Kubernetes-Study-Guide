# Kubernete's DNS 




In earlier videos, we demoed pod-to-pod communication by running curl from a cent-os pod to an apache pod by using the apache pod's ip address. However ip addresses are prone to changing, so it's better to use domain names instead. Luckily most Kubernetes installer tools, such as minikube, sets up DNS for you, by installing a DNS software called CoreDNS. You can then set dns entries in coredns by creating service objects.


This domain name is made up of a few parts:

- the service's name
- namespace name
- the base domain name. 





This kubernetes service ip address will actually never change. That's because this ip address is hardcoded into the kube clusters configuration files. The only way to change this ip address is by deliberately going into your kube cluster's config files and then changing it there. 


By the way, you can control the content of your pod's resolv.conf



## Reference
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/ (talks about how to customise pods resolv.conf file - should be covered in a seperate video)

https://github.com/kubernetes/dns

https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/

## Notes
no need to cover `kubectl get endpoints` here. Can do that as part of deployments demo. 

