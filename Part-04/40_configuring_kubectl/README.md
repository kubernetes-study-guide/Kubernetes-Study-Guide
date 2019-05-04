# Create a ~/.kube/config file manually

In this article we'll recreate the minikubes kubectl context from scratch. However we'll refer to the existing kubectl configs to get settings, e.g. paths to tls files. Let's rename our kubectl config file so that we can start from scratch:


```bash
$ cd ~/.kube/

$ cat config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key

$ mv config config-orig

$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

kubectl is now broken, so let's regenerate it again. You can write the `~/.kube/config` manually using a text editor, however that is prone to causing syntax errors. So it's best to create and manage the `~/.kube/config` file content using the `kubectl config` subcommand. As you saw earlier, there's 4 sections we need to add to our config file:

1. cluster
2. user credentials
3. context (which should also specify namespace to use)
4. currently active context (only one context at any one time)

So let's start by creating the cluster:


```bash
$ kubectl config set-cluster minikube-cluster --server=https://192.168.99.110:8443 --certificate-authority=/Users/schowdhury/.minikube/ca.crt  # --embed-certs=true
Cluster "minikube-cluster" set.
```

This results in:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
contexts: []
current-context: ""
kind: Config
preferences: {}
users: []
```

Notice that my `~/.kube/config` file now references the ca.crt. So our config file now dependes on this ca.crt file to be present at this location at all time. If you want to avoid having these dependencies, then you can use the `--embed-certs=true` flag. This will end up copying and pasting the ca.crts content into the config file in base64 encoded form. If we now retry:

```bash
$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

That's because:

```bash
$ kubectl config current-context
error: current-context is not set
```

So lets create a context:

```bash
$ kubectl config set-context minikube-default-admin --cluster=minikube-cluster
Context "minikube-default-admin" created.
```

This results in:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
contexts:                    # now we have this context present
- context:                            
    cluster: minikube-cluster
    user: ""
  name: minikube-default-admin        # I've used {clustername}-{namespace-name}-{username} naming convention
current-context: ""
kind: Config
preferences: {}
users: []

```

However the error message is still the same as before:


```bash
$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

That's becuase we need to tell kubectl to use this context:

```bash
$ kubectl config current-context
error: current-context is not set

$ kubectl config use-context minikube-default-admin
Switched to context "minikube-default-admin".

$ kubectl config current-context
minikube-default-admin
```

The `~/.kube/config` now looks like:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
contexts:
- context:
    cluster: minikube-cluster
    user: ""
  name: minikube-default-admin
current-context: minikube-default-admin           # this line is now populated
kind: Config
preferences: {}
users: []
```

Now if we try again, we get a password prompt:

```bash
$ kubectl get nodes
Please enter Username:
```

Now let's add user credentials and then associate it with our context:

```bash
$ kubectl config set-credentials admin-user --client-key=/Users/schowdhury/.minikube/client.key --client-certificate=/Users/schowdhury/.minikube/client.crt
User "admin-user" set.

$ kubectl config set-context minikube-default-admin --cluster=minikube-cluster --user=admin-user
Context "minikube-default-admin" modified.
```

now looks like:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
contexts:
- context:
    cluster: minikube-cluster
    user: admin-user                      # new user now associated to our context
  name: minikube-default-admin
current-context: minikube-default-admin
kind: Config
preferences: {}
users:
- name: admin-user                 # new user created here
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
```

Now let's retry:

```bash
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   40h   v1.14.0
```

This worked, even though we haven't specified what namespace we want to work with. 

```bash
$ kubectl config get-contexts
CURRENT   NAME                     CLUSTER            AUTHINFO   NAMESPACE
*         minikube-default-admin   minikube-cluster   minikube
```

the kubectl defaults to the 'default' namespace, if we don't specify otherwise. But it's best practice to explicitly set the namespace as well:

```bash
$ kubectl config set-context minikube-default-admin --cluster=minikube-cluster --user=admin-user --namespace=default
Context "minikube-default-admin" modified.
```

This results in:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube-cluster
contexts:
- context:
    cluster: minikube-cluster
    namespace: default                    # this line now added
    user: minikube
  name: minikube-default-admin
current-context: minikube-default-admin
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
```

which now results in:

```bash
$ kubectl config get-contexts
CURRENT   NAME                     CLUSTER            AUTHINFO   NAMESPACE
*         minikube-default-admin   minikube-cluster   minikube   default
```

Note: you can't create 'users' in kubernetes. They are [handled using authentication plugins externally](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#users-in-kubernetes). 


To summarise, we ran all of the following:

```bash 
# Create the cluster
kubectl config set-cluster minikube-cluster --server=https://192.168.99.110:8443 --certificate-authority=/Users/schowdhury/.minikube/ca.crt

# Create the credentail
kubectl config set-credentials admin-user --client-key=/Users/schowdhury/.minikube/client.key --client-certificate=/Users/schowdhury/.minikube/client.crt

# Create the context
kubectl config set-context minikube-default-admin --cluster=minikube-cluster --user=admin-user --namespace=default


# Activate the context
kubectl config use-context minikube-default-admin
```

## usernames

Notice we deviated a little from in this example. In our original config file, the username was 'minikube', but we changed it to 'admin-user' and it still worked. That's because kube-apiserver doesnt use this username, and it's actually something we use for our reference only. Instead kubectl uses the certificates subject's CN name as the username:


```bash
$ openssl x509 -in /Users/schowdhury/.minikube/client.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2 (0x2)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=minikubeCA
        Validity
            Not Before: Apr  6 12:07:23 2019 GMT
            Not After : Apr  6 12:07:23 2020 GMT
        Subject: O=system:masters, CN=minikube-user
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:bb:af:fb:bd:eb:e2:95:6a:46:e0:75:2a:05:dd:
                    38:1e:d0:24:bb:de:8b:3c:2e:2d:52:c0:a1:f6:14:
                    5b:37:7f:7e:fe:5f:40:f4:43:d7:f4:2a:d5:30:35:
                    1c:18:16:01:c5:63:f9:ed:33:06:76:55:01:5f:3b:
                    29:b7:7f:74:53:ce:ed:7e:07:9d:ad:44:b6:8d:6b:
                    21:60:95:0c:30:e8:1f:e9:6a:59:08:11:14:09:39:
                    37:e8:3c:dd:be:d5:3b:0f:64:b3:af:9d:96:8a:ac:
                    11:96:9e:3a:26:81:e4:63:d3:0f:ed:cc:a5:6a:b9:
                    54:e1:7a:16:5c:9c:5a:e5:eb:56:4f:42:cc:8e:12:
                    ac:97:b0:e2:38:0b:a7:48:5b:68:47:ef:eb:58:93:
                    45:f2:2c:2c:70:68:df:6c:0a:fd:6b:b1:1a:a4:30:
                    3a:5d:f3:03:2a:41:51:81:48:6d:4e:66:23:61:8b:
                    2f:96:17:4c:15:11:a2:87:0b:fe:e9:81:1b:df:60:
                    d0:ba:98:3d:8f:a5:36:98:62:25:f5:2d:50:da:00:
                    b1:82:55:82:ef:22:0b:4b:5f:bf:12:87:8f:68:be:
                    15:1c:79:cf:81:6f:27:aa:32:ab:ce:36:b4:88:c8:
                    30:f2:f9:19:a0:ef:3d:88:2a:50:10:10:ec:f3:1d:
                    7b:c1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
    Signature Algorithm: sha256WithRSAEncryption
         64:6c:ef:8f:8c:3e:42:c4:37:83:46:51:52:b6:ab:48:22:b9:
         7e:a5:9e:ed:95:3c:57:ba:b8:49:5c:53:d2:25:10:15:b2:b2:
         11:a5:f6:49:90:25:ca:0b:e9:c0:2a:c6:3f:cc:62:a4:77:12:
         67:c3:5e:95:7a:80:50:67:05:ef:a8:b9:7c:72:4c:10:1a:92:
         89:67:56:22:0e:59:59:84:97:6f:79:86:81:43:a3:d1:dd:58:
         af:c5:f2:56:e4:01:d8:de:bd:bd:e7:c7:67:57:09:e7:2b:89:
         d9:c2:2f:7f:fa:75:b1:66:76:99:ab:82:a9:f9:fe:2c:6c:ea:
         ca:e6:c0:7d:d1:af:0f:95:66:e2:8a:f2:95:2a:19:3d:44:db:
         d2:cf:92:80:f8:50:71:d7:f3:bb:c5:8f:db:26:04:01:8c:fd:
         8d:75:26:7a:e1:6a:aa:10:8d:bc:d0:b9:75:57:04:a9:be:a0:
         39:dd:a0:02:12:d2:52:50:86:c2:8f:dc:c8:69:8b:f0:57:78:
         b9:db:1a:8c:44:5d:90:01:07:a9:a0:bd:ad:26:b4:ab:c0:c6:
         e9:2c:3e:cc:39:a9:f0:ab:08:87:f0:e2:9d:5c:64:5e:5f:0b:
         c2:4d:28:67:94:31:37:3a:b9:23:fa:15:3e:79:2f:53:f5:1c:
         e6:f9:3b:61
```


Here, the username that api-server authenticates against is 'minikube-user'. The Organisation setting, which in this case is set to 'system:masters', is also used by kube-apiserver to work out what permission the user has. 

Kubernetes doesn't store info about valid users, i.e. there is no such thing as:

```bash
kubectl get users
```

Instead Kubernetes has various [authentication plugins](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authentication-strategies) available to handle this instead. In this example we used [X509 Client Certs](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#x509-client-certs). So if the crt is signed by the kubecluster's ca, then the username embedded inside it becomes a valid username. 

## Reference


[https://medium.com/@awkwardferny/configuring-certificate-based-mutual-authentication-with-kubernetes-ingress-nginx-20e7e38fdfca](https://medium.com/@awkwardferny/configuring-certificate-based-mutual-authentication-with-kubernetes-ingress-nginx-20e7e38fdfca)

[https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)

[https://github.com/kubernetes-sigs/kubespray/issues/257](https://github.com/kubernetes-sigs/kubespray/issues/257)

[https://ahmet.im/blog/mastering-kubeconfig/](https://ahmet.im/blog/mastering-kubeconfig/)

[https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease](https://learnk8s.io/blog/kubectl-productivity/#4-switch-between-clusters-and-namespaces-with-ease)
