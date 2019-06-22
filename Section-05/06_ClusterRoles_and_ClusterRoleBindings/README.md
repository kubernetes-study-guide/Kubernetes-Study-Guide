# ClusterRole and ClusterRoleBindings


Compared to Roles and RoleBindings, these can be easier to setup. Mainly because a Kubernetes install comes with a set of default:

```sh
$ kubectl get clusterrole
NAME                                                                   AGE
admin                                                                  2d22h
calico-node                                                            2d22h
cluster-admin                                                          2d22h
edit                                                                   2d22h
system:aggregate-to-admin                                              2d22h
...

$ kubectl get clusterrolebindings -o wide
NAME                                                   AGE     ROLE                                                                               USERS                            GROUPS                                            SERVICEACCOUNTS
calico-node                                            2d22h   ClusterRole/calico-node                                                                                                                                               kube-system/calico-node
cluster-admin                                          2d22h   ClusterRole/cluster-admin                                                                                           system:masters
...
```

In our case we'll use one of the existing ones to give Lisa full admin priveleges:


```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: lisa-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin     # here we used one of the ready-made clusterroles
subjects:
  - kind: User            # note we also used 'User' this time just as a demo. 
    apiGroup: rbac.authorization.k8s.io
    name: lisa
```

This ends up as:

```bash
# kubectl get clusterrolebindings lisa-admin -o wide
NAME         AGE     ROLE                        USERS   GROUPS   SERVICEACCOUNTS
lisa-admin   4m37s   ClusterRole/cluster-admin   lisa
```

This now gives Lisa full admin access:

```bash
$ kubectl config get-contexts
CURRENT   NAME                     CLUSTER             AUTHINFO         NAMESPACE
*         codingbee-default-lisa   codingbee-cluster   codingbee-lisa   default


$ kubectl get nodes
NAME           STATUS     ROLES    AGE   VERSION
kube-master    Ready      master   3d    v1.14.0
kube-worker1   Ready      <none>   3d    v1.14.0


$ kubectl get pods --namespace=kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
calico-node-78d5g                     2/2     Running   4          3d
calico-node-8k76w                     2/2     Running   4          3d
calico-node-97f2b                     2/2     Running   0          3d
coredns-fb8b8dccf-m64m9               1/1     Running   2          3d
...
```
