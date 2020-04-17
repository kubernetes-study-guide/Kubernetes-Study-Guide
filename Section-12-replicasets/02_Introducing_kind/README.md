# Course notes - Introducing kind

```
brew install kind
```

Then:

```
mkdir ~/.kind
```

Then:

```
curl https://raw.githubusercontent.com/kubernetes-sigs/kind/master/site/content/docs/user/kind-example-config.yaml -o ~/.kind/kind1-config.yaml
```

Then create your cluster:

```
kind create cluster --name kind1 --config ~/.kind/kind1-config.yaml
```

Then you can do:

```
kubectl get nodes
NAME                  STATUS     ROLES    AGE   VERSION
kind1-control-plane   Ready      master   57s   v1.17.0
kind1-worker          Ready      <none>   30s   v1.17.0
kind1-worker2         Ready      <none>   24s   v1.17.0
kind1-worker3         NotReady   <none>   25s   v1.17.0
```