# NetworkPolicies

So far we have seen that every pod is assigned it's own IP address and all the pods in a cluster can reach each other using these IP addresses, and consequently via services too. That's not something that you might want, e.g. a wordress pod usng a mysql pod as it's database, in that scenario, only the wordpress pods needs access to the mysql pod.

So to fix this we need to set up firewalls in our kubernetes networking, and that's done by creating NetworkPolicy objects. NetworkPolicies objects are used for restricting a pod's inbound (aka ingress) and outbound (aka egress) network traffic.

Let's show how this works by going through a demo. In order to run this demo in minikube, you need to create a new minikube cluster with the [minikube cni flags](https://kubernetes.io/docs/setup/minikube/#containerd):

```bash
minikube start --network-plugin=cni --enable-default-cni
```





For this demo we'll start by creating the following pods service:


```bash
$ kubectl apply -f configs/svc-ClusterIP-httpd.yml
service/svc-clusterip-httpd created
$ kubectl apply -f configs/dep-httpd.yml
deployment.apps/dep-httpd created
$ kubectl apply -f configs/pod-centos.yml
pod/pod-centos created

$ kubectl get pods -o wide --show-labels
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES   LABELS
dep-httpd-fdcdd697f-4lcdj   1/1     Running   0          52s   172.17.0.8    minikube   <none>           <none>            component=httpd_webserver,pod-template-hash=fdcdd697f
dep-httpd-fdcdd697f-55rkb   1/1     Running   0          52s   172.17.0.7    minikube   <none>           <none>            component=httpd_webserver,pod-template-hash=fdcdd697f
dep-httpd-fdcdd697f-dv9bz   1/1     Running   0          52s   172.17.0.9    minikube   <none>           <none>            component=httpd_webserver,pod-template-hash=fdcdd697f
pod-centos                  1/1     Running   0          43s   172.17.0.10   minikube   <none>           <none>            app=healthchecker

```

After which we can see that our pod-centos pod is successfully hitting the httpd pods:

```bash
$ kubectl logs pod-centos
You've hit - dep-httpd-fdcdd697f-55rkb
You've hit - dep-httpd-fdcdd697f-dv9bz
You've hit - dep-httpd-fdcdd697f-4lcdj
You've hit - dep-httpd-fdcdd697f-55rkb
You've hit - dep-httpd-fdcdd697f-4lcdj
You've hit - dep-httpd-fdcdd697f-55rkb
```

At the moment, its successfully hitting all the pods in this deployment. 

Now we can block pod-centos's access to the httpd pods by creating:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-by-default
  namespace: default
spec:
  podSelector: {}     # This empty map means, apply this network policy to all pods in the given namespace
```

This creates:

```bash
$ kubectl get networkpolicies
NAME              POD-SELECTOR   AGE
deny-by-default   <none>         2m50s
```



## References

[https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)