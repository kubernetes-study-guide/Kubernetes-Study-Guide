We've actually created the minikube cluster as part of the minikube install video. Let's check that it's still running:


```
$ minikube status.
```

We can also get kubectl to perform a health check.

```bash
$ kubectl get componentstatuses
```

Everything looks good here too. Now let's get a list of all the nodes in our kube cluster.

```
kubectl get nodes
```

We can get more info by setting the output flag to wide.

```
kubectl get nodes -o wide
```

here we can only see one node listed, which is expected since minikube only builds single node kube clusters.

