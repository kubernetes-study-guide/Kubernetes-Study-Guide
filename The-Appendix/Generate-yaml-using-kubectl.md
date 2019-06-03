# Create YAML templates using kubectl

Writing yaml files from scratch can be a time consuming process. That's why it's quicker to get kubectl to generate example yaml definitions that you can use as a starting point

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
kubectl create namespace ns-dev -o yaml --dry-run > namespace.yaml
```

## Deployment Templates

```bash
kubectl create deployment deployment-name --image=nginx --dry-run -o yaml > deployment.yaml
```

Note the following also works, but "kubectl run" could get deprecated in future:

```bash
kubectl run nginx --image=nginx --dry-run -o yaml > deployment.yaml
```

## Job Template

```bash
kubectl run job-name --image=nginx --restart=OnFailure --dry-run -o yaml > job.yaml
```

## CronJob Template

```bash
kubectl run cronjob-name  --schedule="*/1 * * * *" --restart=OnFailure --image=busybox --dry-run -o yaml > cronjob.yaml
```

## Secrets

```bash
kubectl create secret generic secret-name --from-literal MysqlRootPassword=password123 --dry-run -o yaml > secret.yaml
```

## bookmarks

https://medium.com/faun/how-to-pass-certified-kubernetes-administrator-cka-exam-on-first-attempt-36c0ceb4c9e

[https://unofficialism.info/posts/unofficial-tips-for-cka-and-ckad-exams/](https://unofficialism.info/posts/unofficial-tips-for-cka-and-ckad-exams/)

[https://cheatsheet.dennyzhang.com/cheatsheet-kubernetes-a4](https://cheatsheet.dennyzhang.com/cheatsheet-kubernetes-a4)