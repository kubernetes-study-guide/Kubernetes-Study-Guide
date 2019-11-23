So far we have touched on nodeport service. However there are several types of service object and nodeport is just one of them. 

- nodeport
- clusterip 
- loadbalancer
- ingress

the key feature that makes nodeport services standout from the other services is that it's used for allowing outside traffic into the cluster. 

however in the case of running minikube on our local workstation. We can only access any exposed nodeport service from our macbook, however that's not the case in real world scenarios. 

One possible real world scenario, your cluster is made of EC2 VMs running on the aws cloud platform. You have also registered your own domain using a service like Godaddy, and you use AWS route53 to configure your DNS to point to a loadbalancer, forwards traffic to your worker nodes. 

{lots of powerpoint slide here to show above diagram}

In this scenario this works great. although still involves using lots of non standard ports. Which isn't that neat.






However we can access our nodeport from our macbook. 


```
curl http://${minikube ip}:30050
```

We can even curl our nodeport's service name by updating our mac's etc hosts file with a new entry that resolves our nodeport name to our minikube's ip address.

cat /etc/host
nodeport-name   minikube-ip-address.

