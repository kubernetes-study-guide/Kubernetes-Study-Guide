# Create YAML templates using kubectl

Here are the common examples:

## Pod Template

```bash
kubectl run pod-name --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
```

## Nodeport Service Template
```bash
kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 --dry-run -o yaml > service.yaml
```

## Namespace Template

```bash
kubectl create namespace ns-dev -o yaml --dry-run > namespace-boilerplate.yaml
```

