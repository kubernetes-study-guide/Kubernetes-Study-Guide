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


You can attach a serviceaccount to your pod using the `pods.spec.serviceAccountName` setting. If this setting is omitted then the 'default' sa get's attached instead.

The service account get attached to a pod in the form of a secret volume:

```bash
# kubectl edit pods pod-httpd
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
# kubectl exec pod-httpd -it -- /bin/bash

root@pod-httpd:/usr/local/apache2# cd /var/run/secrets/kubernetes.io/serviceaccount

root@pod-httpd:/var/run/secrets/kubernetes.io/serviceaccount# ls -l
total 0
lrwxrwxrwx 1 root root 13 Apr  7 18:20 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Apr  7 18:20 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Apr  7 18:20 token -> ..data/token

root@pod-httpd:/var/run/secrets/kubernetes.io/serviceaccount# cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3Nlc
...
```

So basically pods authenticates itself with the kube-apiserver by using the [token](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) approach. 

The default service account is actually quite powerful. it's gives a pod control over everything at the namespace level. 
