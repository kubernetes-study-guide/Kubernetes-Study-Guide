# ClusterRole and ClusterRoleBindings


Compared to Roles and RoleBindings, these can be easier to setup. Mainly because a Kubernetes install comes with a set of default:

```bash
# kubectl get clusterrole
NAME                                                                   AGE
admin                                                                  2d22h
calico-node                                                            2d22h
cluster-admin                                                          2d22h
edit                                                                   2d22h
system:aggregate-to-admin                                              2d22h
...

# kubectl get clusterrolebindings -o wide
NAME                                                   AGE     ROLE                                                                               USERS                            GROUPS                                            SERVICEACCOUNTS
calico-node                                            2d22h   ClusterRole/calico-node                                                                                                                                               kube-system/calico-node
cluster-admin                                          2d22h   ClusterRole/cluster-admin                                                                                           system:masters
...
```

You can use existing ones if they meet your needs. 