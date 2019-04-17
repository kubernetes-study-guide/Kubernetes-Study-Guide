# NetworkPolicies

So far we have seen that every pod is assigned it's own IP address and all the pods in a cluster can reach each other using these IP addresses, and consequently via services too. That's not something that you might want, e.g. a wordress pod usng a mysql pod as it's database, in that scenario, only the wordpress pods needs access to the mysql pod.

So to fix this we need to set up firewalls in our kubernetes networking, and that's done by creating NetworkPolicy objects. NetworkPolicies objects are used for restricting a pod's inbound (aka ingress) and outbound (aka egress) network traffic.


Let's show how this works by going through a demo. In order to run this demo in minikube, you need to create a new minikube cluster with the [minikube cni flags](https://kubernetes.io/docs/setup/minikube/#containerd):

```bash
minikube start --network-plugin=cni --enable-default-cni
```

However even with this I couldn't get networkpolicies to work so I have used the vagrant-kubeadm environment. 


## Create dummy test environment (eg0-initial-setup-for-demo)
We'll start by creating a dummy environment for this demo:

```bash
$ kubectl apply -f configs/eg0-initial-setup-for-demo/
deployment.apps/dep-httpd created
namespace/codingbee created
pod/pod-centos created
service/svc-clusterip-caddy created
service/svc-clusterip-httpd created
deployment.apps/dep-caddy created
```

You might get a 'namspace not found' error when running this command. That's because of a race condition, it's trying to create pod in a namespace that's still in the process of being created. If so then run the above command again. 

After which, you should find the following pods and services:


```bash
$ # kubectl get pods -o wide --show-labels
NAME                         READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES   LABELS
dep-httpd-6f45c8fd4c-cf4vw   1/1     Running   0          8m4s    192.168.1.25   kube-worker1   <none>           <none>            component=httpd_webserver,pod-template-hash=6f45c8fd4c
dep-httpd-6f45c8fd4c-dmrtk   1/1     Running   0          8m4s    192.168.1.23   kube-worker1   <none>           <none>            component=httpd_webserver,pod-template-hash=6f45c8fd4c
dep-httpd-6f45c8fd4c-jmv5p   1/1     Running   0          8m4s    192.168.1.24   kube-worker1   <none>           <none>            component=httpd_webserver,pod-template-hash=6f45c8fd4c
pod-curl-client              1/1     Running   0          2m22s   192.168.1.29   kube-worker1   <none>           <none>            app=curl_client

$ kubectl get service -o wide
NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes            ClusterIP   10.96.0.1     <none>        443/TCP   14m     <none>
svc-clusterip-httpd   ClusterIP   10.96.186.3   <none>        80/TCP    9m25s   component=httpd_webserver
```

We also ended creating the following namespace alongside the following pods (via deployments) and services: 

```bash
# kubectl get namespaces codingbee
NAME        STATUS   AGE
codingbee   Active   95m
# kubectl get service --namespace=codingbee
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
svc-clusterip-caddy   ClusterIP   10.101.67.30   <none>        80/TCP    20m
# kubectl get pods --namespace=codingbee
NAME                         READY   STATUS    RESTARTS   AGE
dep-caddy-5b6ff8d5fb-dbw68   1/1     Running   0          21m
dep-caddy-5b6ff8d5fb-tjgp4   1/1     Running   0          21m
```

The pod-curl-client is set up to perioding ping both the caddy and httpd services:


```bash
$ kubectl logs pod-curl-client
#############################################################
Tue Apr 16 20:12:25 UTC 2019
Attempting connection to httpd:
You've hit httpd - dep-httpd-6f45c8fd4c-jmv5p
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68
#############################################################
Tue Apr 16 20:12:35 UTC 2019
Attempting connection to httpd:
You've hit httpd - dep-httpd-6f45c8fd4c-dmrtk
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-tjgp4
#############################################################
Tue Apr 16 20:12:45 UTC 2019
Attempting connection to httpd:
You've hit httpd - dep-httpd-6f45c8fd4c-dmrtk
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68
...
```

At the moment, its successfully hitting all the in both httpd and caddy deployments. Also we can curl from caddy->httpd, and vice versa:

```bash
$ kubectl exec dep-httpd-6f45c8fd4c-cf4vw -it -- bash
root@dep-httpd-6f45c8fd4c-cf4vw:/usr/local/apache2# curl http://svc-clusterip-caddy.codingbee.svc.cluster.local
You've hit caddy - dep-caddy-5b6ff8d5fb-tjgp4

$ kubectl exec dep-caddy-5b6ff8d5fb-dbw68 -it --namespace=codingbee -- sh
/srv # curl http://svc-clusterip-httpd.default.svc.cluster.local
You've hit httpd - dep-httpd-6f45c8fd4c-dmrtk
```

We performed the test using the dns entries, but it would have also worked if we tried connecting using a pod's IP address. So essentially all pods can reach each other. Note, none of the pods can't initiate a connection to the pod-curl-client pod, only because pod-curl-client doesn't contain a service that's listening on a port. 



## Block all traffic

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
# kubectl get networkpolicies
NAME                   POD-SELECTOR   AGE
block-all-by-default   <none>         23s
```

This ends up blocking all pod-to-pod traffic in the default namespace. Here's what the latest log entries now look like:

```bash
$ kubectl logs pod-curl-client
...
#############################################################
Tue Apr 16 20:55:12 UTC 2019
Attempting connection to httpd:
curl: (28) Connection timed out after 10002 milliseconds
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68
#############################################################
Tue Apr 16 20:55:32 UTC 2019
Attempting connection to httpd:
curl: (28) Connection timed out after 10033 milliseconds
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-tjgp4
```

As you can see, our client pod can still reach the caddy pods because the networkpolicy is only applied for inbound traffic for pods in the default namespace. Also the httpd pods can no longer reach other pods in it's own deployment:

```bash
$ kubectl exec dep-httpd-6f45c8fd4c-cf4vw -it -- bash
root@dep-httpd-6f45c8fd4c-cf4vw:/usr/local/apache2# curl --silent --show-error --connect-timeout 10 http://svc-clusterip-httpd.default.svc.cluster.local
curl: (28) Connection timed out after 10000 milliseconds
```


## PodSelector demo (eg2-podSelector-demo)


Now let's create the following:


```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-to-httpd 
  namespace: default
spec:
  podSelector:
    matchLabels:
      component: httpd_webserver    # This np gets applied to all pods with this label. 
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - ipBlock:               
          cidr: 172.17.0.0/16
          except:
          - 172.17.1.0/24
      - podSelector:                  # Here we're saying that any pods with this label is permitted access. 
          matchLabels:
            app: curl_client 
      ports:         # notice here we can apply port level restriction as well
        - protocol: TCP
          port: 80
```

The 'ipBlock' is another approach, but this bit doesn't get used for anything here. But it's here just to show other possibilities. Also the 'ports' section appears to need to be indented a bit more than I thoughght would be necessary, not sure why that is. 



We now have a new NetworkPolicy:

```bash
$ kubectl get networkpolicies
NAME                   POD-SELECTOR                AGE
access-to-httpd        component=httpd_webserver   3s
block-all-by-default   <none>                      12h
```

Here we see that the this rule is this time attached to specif pod. 
After which we end up with:

```bash
kubectl logs pod-curl-client -c cntr-centos
...
#############################################################
Tue Apr 16 21:12:34 UTC 2019
Attempting connection to httpd:
You've hit httpd - dep-httpd-6f45c8fd4c-jmv5p
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68
#############################################################
Tue Apr 16 21:12:44 UTC 2019
Attempting connection to httpd:
You've hit httpd - dep-httpd-6f45c8fd4c-jmv5p
Attempting connection to caddy:
You've hit caddy - dep-caddy-5b6ff8d5fb-tjgp4
```


Now access is working, that's becuase access rules overrides deny rules. However other pods in the cluster don't have access:


```bash
$ kubectl exec dep-httpd-6f45c8fd4c-cf4vw -it -- bash
root@dep-httpd-6f45c8fd4c-cf4vw:/usr/local/apache2# curl --silent --show-error --connect-timeout 5 http://svc-clusterip-httpd.default.svc.cluster.local
curl: (28) Resolving timed out after 5571 milliseconds


$ kubectl exec dep-caddy-5b6ff8d5fb-dbw68 -it --namespace=codingbee -- sh
/srv # curl --silent --show-error --connect-timeout 5 http://svc-clusterip-httpd.default.svc.cluster.local
curl: (28) Connection timed out after 5001 milliseconds
/srv #
```

## Permit access using namespaceSelector (eg3-namespaceSelector)

Here's an example of giving a whole namespace access to your pods, by editing an existing networkpolicy:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-to-httpd 
  namespace: default
spec:
  podSelector:
    matchLabels:
      component: httpd_webserver  
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - ipBlock:               
          cidr: 172.17.0.0/16
          except:
          - 172.17.1.0/24
      - namespaceSelector:   # here we grant access at the namespace level
          matchLabels:
            project: codingbee
      - podSelector:
          matchLabels:
            app: curl_client 
      ports:
        - protocol: TCP
          port: 80
```

This results in granting access to all pods from all namespaces that has the matching label. The codingbee namespace has this label.

```bash
$ kubectl get namespaces --show-labels
NAME              STATUS   AGE   LABELS
codingbee         Active   15h   project=codingbee
default           Active   16h   <none>
kube-node-lease   Active   16h   <none>
kube-public       Active   16h   <none>
kube-system       Active   16h   <none>
```

Now the caddy pods have access to httpd too:

```bash
$ kubectl exec dep-caddy-5b6ff8d5fb-dbw68 -it --namespace=codingbee -- sh
/srv # curl --silent --show-error --connect-timeout 10 http://svc-clusterip-httpd.default.svc.cluster.local
You've hit httpd - dep-httpd-6f45c8fd4c-cf4vw
```

## Egress demo

Now let's do an egress demo. At the moment our pod-curl-client can reach the caddy pod, via it's service:

```bash
$ kubectl get svc -o wide --namespace=codingbee
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-clusterip-caddy   ClusterIP   10.101.67.30   <none>        80/TCP    16h   component=caddy_pod

$ kubectl get pods -o wide --namespace=codingbee --show-labels
NAME                         READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES   LABELS
dep-caddy-5b6ff8d5fb-dbw68   1/1     Running   0          16h   192.168.1.27   kube-worker1   <none>           <none>            component=caddy_pod,pod-template-hash=5b6ff8d5fb
dep-caddy-5b6ff8d5fb-tjgp4   1/1     Running   0          16h   192.168.1.28   kube-worker1   <none>           <none>            component=caddy_pod,pod-template-hash=5b6ff8d5fb

$ kubectl exec pod-curl-client -it -- bash

[root@pod-curl-client /]$ curl --silent --show-error --connect-timeout 1 http://svc-clusterip-caddy.codingbee.svc.cluster.local
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68

[root@pod-curl-client /]$ curl --silent --show-error --connect-timeout 1 http://192.168.1.27:2015
You've hit caddy - dep-caddy-5b6ff8d5fb-dbw68
```

When using the ip, we had to specify the port, 2015, since that's the port the caddy pods are listening to. Now let's say we want the caddy pods to continue to accept traffic from port 2015, but we want to prevent all port 2015 outbound traffic from the pod-curl-client pod. In that case we can create the following netpool:

```yaml

```
Note, i could only get the ipblock egress to work. I can't podselector to work. 



## References

[https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

[https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)