# Tips of writing YAML files

When writing yaml files, You can refer to the [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/) to work out what to set for your chosen object type (kind). This reference doc is really useful displays example yaml content for your chosen kind. This link is for version v1.14. But in your case you need go to the link, that's specified with the Major and Minor tag of Server Version in:

```bash
$ kubectl version --short
Client Version: v1.13.4
Server Version: v1.14.0
```

This shows that the kubectl binary we have installed locally on our macbook is v1.13.4, whereas the kubecluster our kubectl command is talking to is version v1.14.0.

You can also view the api reference data from the cli, using the 'explain' subcommand:

```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.
...
```

You can get a high-level overview of the entire yaml structure:

```bash
kubectl explain pod --recursive | less
```

And you can drill down like this:

```bash
$ kubectl explain pods.spec.containers.env --recursive
KIND:     Pod
VERSION:  v1

RESOURCE: env <[]Object>

DESCRIPTION:
     List of environment variables to set in the container. Cannot be updated.
...
```


## View entire yaml definition including implicit values

What if you have a pod, e.g. called pod-httpd, but lost it's yaml descriptor file? In that case you can regenerate the yaml descriptor from the existing pod:

```bash
kubectl get pod pod-httpd -o yaml > regenerated-descriptor.yaml
```

## References

[kubernetes api concepts](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
[kubernetes api reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/)