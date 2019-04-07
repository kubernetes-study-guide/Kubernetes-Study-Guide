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
  - apiGroups: ["v1"] 
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  - apiGroups: ["v1"] 
    resources: ["services"]
    verbs: ["get", "watch", "list"]
```

This creates:


```bash
# kubectl get role -o wide
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
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: role-read
subjects:
  - kind: Group
    name: employee
```

Here we attached our role to a group. Note you can associate multiple users/groups to a single role. This creates:

```bash
# kubectl get rolebindings -o wide
NAME      AGE   ROLE             USERS   GROUPS     SERVICEACCOUNTS
rb-read   8s    Role/role-read           employee
```

Also notice that in kubernetes we don't create 'groups'. the concept of groups works in a similar way to selectors+labels. The tls certificate specifies a Organisation in the subject section, which kubernetes uses as a group's name, and matches to rolebindings, which have a matching name in `rolebindings.subjects.name`.




# Reference

[# https://kubernetes.io/docs/reference/access-authn-authz/rbac/](# https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

[https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d](https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d)

