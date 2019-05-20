

By the way, here's a way to generate a namespace boilerplate template:

```bash
kubectl create namespace ns-dev -o yaml --dry-run > namespace-boilerplate.yaml
```