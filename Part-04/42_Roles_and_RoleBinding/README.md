# RBAC

This article follows on from the previous article where we created a new user, called Lisa. We want to give List view access to pods. 


To do this we need to make use of Role objects. At the moment we don't have any roles, so let's create a new role:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: role-read
  namespace: default    # This is mandatory, if omitted then 'default' is used
rules:
  - apiGroups: [""]      # find this by running the 'kubectl api-resources' command
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  - apiGroups: [""]             # find this by running the 'kubectl api-resources' command
    resources: ["services"]
    verbs: ["get", "watch", "list"]
```

This creates:


```bash
$ kubectl get role -o wide
NAME            AGE
employee-role   74s
```

This is a permission, but we now need to attach this permission to a user or group. We do this by creating a rolebinding object:

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-read  
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io     # this is a fixed string
  kind: Role
  name: role-read
subjects:
  - kind: Group
    apiGroup: rbac.authorization.k8s.io    # this is a fixed string
    name: employee
```

You can associate multiple users/groups to a single role. Here we attached our role to the 'employee' group:

```bash
$ kubectl get rolebindings -o wide
NAME      AGE   ROLE             USERS   GROUPS     SERVICEACCOUNTS
rb-read   8s    Role/role-read           employee
```

Also notice that in kubernetes we don't create a group called 'employee'. the concept of groups works in a similar way to selectors+labels. The tls certificate specifies a Organisation in the subject section, which kubernetes uses as a group's name, and matches to rolebindings, which have a matching name in `rolebindings.subjects.name`.

Note: You can also attach ServiceAccounts to roles. We'll cover ServiceAccounts later. 

After that, Lisa will now be able list pods and services, when she runs:

```bash
$ kubectl config get-contexts
CURRENT   NAME                     CLUSTER             AUTHINFO         NAMESPACE
*         codingbee-default-lisa   codingbee-cluster   codingbee-lisa   default

$ kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP          2d22h
svc-nodeport-httpd   NodePort    10.109.105.25   <none>        3050:31000/TCP   32s

$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          35s
```

However these priveleges are limited to the namespace we specified in our role/rolebinding definitions, so you will get error messages if you try to query other namespaces:

```bash
$ kubectl get pods --namespace=kube-system
Error from server (Forbidden): pods is forbidden: User "lisa" cannot list resource "pods" in API group "" in the namespace "kube-system"
```

Also roles+rolebindings can't be used to set up access for things that live outside namespaces, e.g. nodes:

```bash
$ kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "lisa" cannot list resource "nodes" in API group "" at the cluster scope
```

To overcome these limitiation, you can create cluster wide roles and rolebindings, called CluserRoles and ClusterRolebindings. Which we'll cover next. 


## Reference

[https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

[https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d](https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d)

