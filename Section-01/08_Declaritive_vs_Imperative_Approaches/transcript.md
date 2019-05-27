# Imperative vs Declaritive approaches

There's generally two ways to use kubectl when managing your kubernetes objects (pods, services,...etc), they are the 'Declaritive' and 'Imperative' approaches.

The Declaritive approach is about using the `kubectl apply` command when creating objects. That's the approach we took in our hello-world demo. The declarative approach therefore depends on us first creating the yaml files before we can create the objects. 

The imperative approach involves using any of the verb based commands, such as:

- `kubectl run`
- `kubectl create`
- ...and etc

Now, Here's an example of how to create a pod using kubectl run:

```bash
kubectl run pod-httpd --image=httpd --labels="app=apache_webserver" --restart=Never
kubeclt get pods
kubeclt delete pods
```



You can also feed yaml files into some of the imperative commands, such as create and delete:

```bash
$ kubectl create -f configs/
pod/pod-httpd created

$ kubectl delete -f configs/
pod "pod-httpd" deleted
```

In the documentation it says that the [Declaritive technique is the recommended approach when working in production](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/). However in reality it depends on the the task you want to perform. For example you should take the declarative approach when creating pods, so that your pod's yaml files are documentend and version controlled in Git. On the otherhand, we use the imperative approach to [delete objects](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/declarative-config/#how-to-delete-objects).


One very handy feature with imperative commands is that you can use them to generate example yaml files. You can then use these files as boilerplates to create your own yaml files:

```bash
kubectl run pod-httpd --image=httpd --restart=Never --dry-run -o yaml > pod.yaml

kubectl create service nodeport svc-nodeport-httpd --node-port=31000 --tcp=3050:80 > service.yaml
```

In the CKA Exam, You're not allowed to copy+paste more than a couple of lines at a time. That's why using imperative commands like these will be a massive time saver and definitely worth memorising. We'll include a complete list of these commands in the appendix. 