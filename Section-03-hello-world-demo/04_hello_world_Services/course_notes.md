 NodePort services are quite crude and isn't recommended for production, we'll explain why that is in a later video, but we're using going to create a nodePort service in this demo because it's relatively easy to understand.





 Ok we've now seen the nodeport service in action, but what's actually going on behind the scenes? Why do we have to use 3 different port numbers?

Let me pull up the following diagram to help explain what's going on. This diagram shows a kubecluster that has a total of 3 worker nodes.

When you create pod, it get's created on a particular worker node. But services on the other hand are cluster wide objects, which means that a single service object      span across all worker nodes. 

When you create a nodeport service, kubernetes ends up configuring each worker node at teh OS level to start listening on the Nodeport Number. It also sets up networking so that any incoming nodeport traffic get's forwarded to the nodeport service. In order for that to work, the nodeport service itself also needs to be listening on a port number. When the nodeport service receives traffic it then forwards it to a pod with label that matches the selector. In this example we have 2 pods that has the matching label this, In this scenario the nodeport service will also act as a loadbalancer to these pods. 

Also the nodeport service needs to know which port number on the pod (i.e. targetport) to forward the traffic too. That's where the targetport comes in. 

Now we also have this curl pod. This is just a dummy pod I've added just for performing some simple tests. In this scenario, our curl-pod can curl to either of these httpd pods using their respective ip addresses and port 80. Or it can curl via the service's ip address, in which case it needs to do it via port 3050. We'll demo that in a later video. 

In our minikube setup, we have a similar setup. It's essentially the same thing but with a single node instead of three, but the core concepts are still the same. 




It would be nice to pull our pod and services at the same time, you can do that like this:

```bash
$ kubectl get pods,services -o wide
```

or you can do:


kubectl get all --all