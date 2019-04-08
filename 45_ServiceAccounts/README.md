# ServiceAccounts

Earlier we saw how we can control an individual's (or group's) permissions, at the [authorisation](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/) stage by using Roles, RolesBindings, ClusterRoles, and ClusterRolesBindings objects.

However you can also use these objects to control access privileges for pods too. That's because pods also goes through the same authentication+authorization process if/when it sets api requests to the kube-api server. 

A pods authentication is done via the use of ServiceAccounts. A ServiceAccount is an identity that's attached to a pod. Each namespace comes with a default serviceaccount:

```bash
# kubectl get serviceaccounts
NAME      SECRETS   AGE
default   1         3d1h

# kubectl get serviceaccounts default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2019-04-04T20:16:47Z"
  name: default
  namespace: default
  resourceVersion: "351"
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: 92fa5b57-5716-11e9-b032-080027ee87c4
secrets:
- name: default-token-z8ffc

# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-z8ffc   kubernetes.io/service-account-token   3      3d1h

```

You can attach a serviceaccount to your pod using the `pods.spec.serviceAccountName` setting. If this setting is omitted then the 'default' sa get's attached instead. Service accounts are attached to a pod in the form of a secrets volume:

```bash
# kubectl edit pods pod-centos 
...
spec:
  volumes:
  - name: default-token-z8ffc
    secret:
      defaultMode: 420
      secretName: default-token-z8ffc
  containers:
    volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: default-token-z8ffc
        readOnly: true
...

```

If you take a look inside one of this pod's containers, you'll find:

```bash
[root@pod-centos /]# cd /var/run/secrets/kubernetes.io/serviceaccount
[root@pod-centos serviceaccount]# ls -l
total 0

lrwxrwxrwx 1 root root 13 Apr  8 08:38 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Apr  8 08:38 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Apr  8 08:38 token -> ..data/token

[root@pod-centos serviceaccount]# cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9...
```

So basically pods authenticates itself with the kube-apiserver by using the [token](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) approach.
```


The default SA isn't attached to any clusterbinding/rolebindings. Which means that it can only authenticate with the kube-apiserver but can't do anything else. To show a successful authentication, we'll test the sa by running a kubectl command inside the pod, first we have to [install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-native-package-management) inside the pod itself:


```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl


[root@pod-centos serviceaccount]# kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
```

Here we can see that we got a forbidden error message, rather than an invalid credentials message, which means we've successfully authenticated. So let's say we want to give this pod fill clusterwide admin privileges, then we can do this by attaching cluster-admin priveleges to the default sa:

```bash
# kubectl create clusterrolebinding cluster-admin-for-default-SA --clusterrole=cluster-admin --serviceaccount=default:default
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-for-default-SA created

# kubectl describe clusterrolebindings cluster-admin-for-default-SA
Name:         cluster-admin-for-default-SA
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind            Name     Namespace
  ----            ----     ---------
  ServiceAccount  default  default
```

Now our pod has full access:

```bash
# kubectl exec pod-centos -it -- /bin/bash
[root@pod-centos /]# kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
pod-centos   1/1     Running   1          11h
pod-httpd    1/1     Running   1          14h
[root@pod-centos /]# kubectl get nodes
NAME           STATUS     ROLES    AGE     VERSION
kube-master    Ready      master   3d12h   v1.14.0
kube-worker1   Ready      <none>   3d12h   v1.14.0
kube-worker2   NotReady   <none>   3d12h   v1.14.0
```


# Create and Use Service Accounts

However this isn't good practice because it means all pods that are using the 'default' SA will also end up with full priveleges. A better approach would be to create create custom service accounts attach the appropriate priveleges, then attach them to the pods that need those privileges.




