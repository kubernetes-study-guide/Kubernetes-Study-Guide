# Service Accounts

In Kubernetes, it's possible to run kubectl commands (e.g. kubectl get pods), from inside pods themselves. In doing so pods have to follow the same [authentications+authorsations](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/) process that individuals are subjected. This also means that you can control a pods access privileges using Roles, RolesBindings, ClusterRoles, and ClusterRolesBindings objects.

## Pod Authentication with the kube-apiserver
When humans use the kubectl command, we establish our identity with the kube-apiserver using TLS certificates. However for non-humans, i.e. pods, the pods identifies itself to the kube-apiserver by using Service Account objects. A ServiceAccount is an identity that's attached to a pod. Each namespace comes with a default serviceaccount:

```bash
$ kubectl get serviceaccounts
NAME      SECRETS   AGE
default   1         3d1h

$ kubectl get serviceaccounts default -o yaml
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

$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-z8ffc   kubernetes.io/service-account-token   3      3d1h

```

You can attach a serviceaccount to your pod using the `pods.spec.serviceAccountName` setting. If this setting is omitted then the 'default' sa get's attached instead. Service accounts are attached to pods via secrets volume:

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

So basically pods authenticates itself with the kube-apiserver by ServiceAccount objects which in turn uses the [token](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) authentication approach. Lets try using this ServiceAccount by running a kubectl command inside a pod, first we have to [install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-native-package-management) inside the pod itself:


```bash
# kubectl exec pod-centos -it -- /bin/bash
[root@pod-centos /]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

[root@pod-centos /]# yum install -y kubectl


[root@pod-centos /]# kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
```

Here we can see that we got a forbidden error message, rather than an invalid credentials message, which means we've successfully authenticated. So let's say we want to give this pod fill clusterwide admin privileges, then we can do this by attaching cluster-admin priveleges to the default sa:

```bash
$ kubectl create clusterrolebinding cluster-admin-for-default-SA --clusterrole=cluster-admin --serviceaccount=default:default
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-for-default-SA created

$ kubectl describe clusterrolebindings cluster-admin-for-default-SA
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
$ kubectl exec pod-centos -it -- /bin/bash
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

Giving this much access privileges to the 'default' service account is not a good idea, because it means all pods that are using the 'default' SA will also end up with full priveleges. A better approach would be to create create another Service Account along with the necessary privileges, and then attach ServiceAccount to your pods as appropriate.

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: basic-access
```

This creates:

```
$ kubectl get sa basic-access
NAME           SECRETS   AGE
basic-access   1         81s

$ kubectl describe sa basic-access
Name:                basic-access
Namespace:           default
Labels:              <none>
Annotations:         kubectl.kubernetes.io/last-applied-configuration:
                       {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"basic-access","namespace":"default"}}
Image pull secrets:  <none>
Mountable secrets:   basic-access-token-nlrc4
Tokens:              basic-access-token-nlrc4
Events:              <none>
```

Also notice that a secret object is created for each ServiceAccount that you create:


```bash
$ kubectl get secrets basic-access-token-nlrc4
NAME                       TYPE                                  DATA   AGE
basic-access-token-nlrc4   kubernetes.io/service-account-token   3      23m
```

This secret contains 3 bits of data, namespace, ca.crt, and token. These data translates to the 3 files that appear inside the pod's mountpoint. Now that we have the ServiceAccount, we can then attach it to a pod:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
  namespace: default
spec: 
  serviceAccountName: basic-access     # We add this line.
  containers:
    - name: cntr-centos
      image: centos
      command: ["/bin/bash", "-c"]
      args:
        - |
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
          while true ; do
            date 
            sleep 10 
          done
```

Note, for convenience we're installing kubectl as part of the startup command. This ends up creating:

```bash
$ kubectl get pods pod-centos -o yaml
spec:
...
  serviceAccount: basic-access
  containers:
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: basic-access-token-nlrc4
      readOnly: true
...
  volumes:
  - name: basic-access-token-nlrc4
    secret:
      defaultMode: 420
      secretName: basic-access-token-nlrc4
...
```

At this point if you exec inside this pod and try to run a kubectl command, then you'll successfully authenticate but will get a 'forbidden' message. That's because we haven't assigned any priveleges to our new new ServiceAccount, basic-access. Now let's say we want to give this SA account just view access to all resources, then we an use one of the out-of-the-box clusterroles that matches that requirement's it called 'view':

```bash
root@kube-master:~/Kubernetes-Study-Guide/45_ServiceAccounts# kubectl describe clusterroles view
Name:         view
Labels:       kubernetes.io/bootstrapping=rbac-defaults
              rbac.authorization.k8s.io/aggregate-to-edit=true
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources                                Non-Resource URLs  Resource Names  Verbs
  ---------                                -----------------  --------------  -----
  bindings                                 []                 []              [get list watch]
  configmaps                               []                 []              [get list watch]
  endpoints                                []                 []              [get list watch]
  events                                   []                 []              [get list watch]
  limitranges                              []                 []              [get list watch]
  namespaces/status                        []                 []              [get list watch]
  namespaces                               []                 []              [get list watch]
  persistentvolumeclaims                   []                 []              [get list watch]
  pods/log                                 []                 []              [get list watch]
  pods/status                              []                 []              [get list watch]
  pods                                     []                 []              [get list watch]
  ...
  ```

So let's create a rolebinding to attach this clusterrole to our service account:


```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-view  
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
  - kind: ServiceAccount
    apiGroup: ""          # found this by running: kubectl api-resources
    name: basic-access
```

This ends up creating:

```bash
root@kube-master:~/Kubernetes-Study-Guide/45_ServiceAccounts# kubectl get rolebindings rb-view -o wide
NAME      AGE   ROLE               USERS   GROUPS   SERVICEACCOUNTS
rb-view   28s   ClusterRole/view                    /basic-access


root@kube-master:~/Kubernetes-Study-Guide/45_ServiceAccounts# kubectl describe rolebindings rb-view
Name:         rb-view
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"RoleBinding","metadata":{"annotations":{},"name":"rb-view","namespace":"default"},"ro...
Role:
  Kind:  ClusterRole
  Name:  view
Subjects:
  Kind            Name          Namespace
  ----            ----          ---------
  ServiceAccount  basic-access
```

Now we'll see that kubectl now works from inside the pod:

```bash
$ kubectl exec pod-centos -it -- /bin/bash
[root@pod-centos /]# kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          3d23h
svc-nodeport-httpd   NodePort    10.109.105.25   <none>        3050:31000/TCP   25h

[root@pod-centos /]# kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
pod-centos   1/1     Running   0          45m
```

However other commands don't work even though they are permitted in the 'view' ClusterRole:

```bash
[root@pod-centos /]# kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "system:serviceaccount:default:basic-access" cannot list resource "nodes" in API group "" at the cluster scope

[root@pod-centos /]# kubectl get PersistentVolumes
Error from server (Forbidden): persistentvolumes is forbidden: User "system:serviceaccount:default:basic-access" cannot list resource "persistentvolumes" in API group "" at the cluster scope

[root@pod-centos /]# kubectl get pods --namespace=kube-system
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:basic-access" cannot list resource "pods" in API group "" in the namespace "kube-system"
```

That's because we've done something a little different this time so that we can demo this behaviour. We referenced a clusterRole in a rolebinding (instead of clusterrolebinding) object. This has the effect of limiting all accesses to the pod's namespace and it also means the pod can't access to things that arent namespace specific, e.g. nodes. This is a handy techniques that saves you the trouble fo creating your own 'Role' objects. 

Another option is that you might want all pods across the cluster to have access to resources in a particular namespace, if so, then you can achieve this by creating a ClusterRoleBinding that associates to a Role. 


## Use curl instead of kubectl

All the api interactions we performed inside the pods so far, we did so using the kubectl command. However you may not want  kubectl installed inside your pods, in that case you can just use curl, since the kube-apiserver is just a REST based API. 
