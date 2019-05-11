# Imperative vs Declaritive approaches

There's essentially two ways to use kubectl for managing your kubernetes objects (pods, services,...etc), they are the [Declaritive](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/declarative-config/) and [Imperative](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/imperative-config/) approaches.

The Declaritive approach is essentially to do with creating your Kubernetes objects using `kubectl apply`. This is the approach we took in our hello-world demo. However, we could have also created these objects using the imperative approach. There are several imperative commands available. For example to imperitavely create our pod we would use the `kubectl run` command:

```bash
kubectl run pod-httpd --image=httpd --restart=Never
```

And to create the service object from the previous example we can do:

```bash
kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80
```

Here are some of the commonly used imperative commands:

- `kubectl run`
- `kubectl expose`
- `kubectl delete`
- `kubectl edit`

There are pros and cons to both approaches. Generally the approach you take depends on the the task you want to perform. However the [Declaritive technique is the recommended approach when working in production](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/). So it's best to take the declaritive approach when it makes most sense. For example you should take the declarative approach when creating pods, so that your pod's yaml files are documentend and version controlled in Git. On the otherhand, we use the imperative approach to [delete objects](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/declarative-config/#how-to-delete-objects).

One very handy feature with imperative commands is that you can use them to create generate example yaml files. You can then use these as boilerplate templates to create your own yaml files:

```bash
kubectl run pod-httpd --image=httpd --restart=Never --dry-run -o yaml > pod.yaml

kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 > service.yaml
```

**CKA exam tip:** Your not allowed to copy+paste more than a couple of lines at a time during the exam. You can write them by hand, but that's time consuming and prone to errors, which will use up even more precious seconds. Which is why it's best to use the imperative commands to create the yaml files for you. We'll include a complete list of these commands in the appendix. 

## References

[https://ansilh.com/03-pods/03-create-pod/](https://ansilh.com/03-pods/03-create-pod/)