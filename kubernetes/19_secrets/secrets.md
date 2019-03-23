# Secrets

In the previous example, our config file had a big flaw in respect to the fact that we stored passwords in plain text. Which is worse if you store your kube object files in a git repo. The recommended approach is to make use of [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/). 

## Creating Secrets

Secrets are just another type of kubernetes objects, so you can create secrets declaratively by creating a yaml file:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets 
data:
  MysqlRootPassword: cGFzc3dvcmQxMjM=    # This entry must be given in base64 format:  echo -n 'password123' | base64
```

In our example we are setting the password to 'password123'. Be careful not to put these files in git repos, since they hold secrets in plain text. base64 isn't encryption, so anyone can decode it by [decoding the kubernetes secret](https://kubernetes.io/docs/concepts/configuration/secret/#decoding-a-secret):

```bash
$ echo 'cGFzc3dvcmQxMjM=' | base64 --decode
password123
```


In fact, for secrets, it could be argued that it's better to create secrets imperatively (i.e. manually from the command line):

```bash
$ kubectl create secret generic mysql-secrets --from-literal MysqlRootPassword=password123
secret/mysql-secrets created


$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-p5k7f   kubernetes.io/service-account-token   3      28m  # this comes included in kube cluster. 
mysql-secrets        Opaque                                1      10s
```

Here, the word 'generic' means refers to the type of secret. The 'generic' type simply meants to create a secret from the contents from local file, directory or literal value. Other options are 'docker-registry' and 'tls'. The '--from-literal' means, use the key=value pair specified on the command line. --from-literal, is only useful for simply secrets, e.g. passwords. But if your secrets comes in the form of a file, e.g. ssh private keys, then use '--from-file' instead, e.g.:

```bash
kubectl create secret generic my-secret --from-file=ssh-privatekey=~/.ssh/id_rsa
```




We've named our secrets object as 'mysql-secrets'. Secrets objects can house multipe key-value pairs for storing secrets. At the moment we're only storing once key-value pair inside this secret. After you've created the secret, you can still view the base64 encoded format of the secret:

```bash
$ kubectl get -o yaml secrets mysql-password
apiVersion: v1
data:
  MYSQL_ROOT_PASSWORD: cGFzc3dvcmQxMjM=
kind: Secret
metadata:
  creationTimestamp: "2019-02-27T12:04:14Z"
  name: mysql-password
  namespace: default
  resourceVersion: "2484"
  selfLink: /api/v1/namespaces/default/secrets/mysql-password
  uid: ccd105fa-3a87-11e9-946d-0800271ef513
type: Opaque

$ echo 'cGFzc3dvcmQxMjM=' | base64 --decode
password123
```

## Using Secrets

There's 2 main ways to inject secrets into your pods:

- Inject secrets as environment variables
- Inject secrets as files into pods - this approach requires creatng [secret volumes](https://kubernetes.io/docs/concepts/storage/volumes/#secret).

### Inject secrets as Environment Variables
Now we modify our pod yaml file so that it now looks like:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-mysql-db
  labels:
    component: mysql_db_server
spec:
  containers:
    - name: cntr-mysql-db
      image: mysql
      env:
        - name: MYSQL_DATABASE
          value: dummy_db
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets      # this is the name of the object that holds one or more key-value pairs. 
              key: MysqlRootPassword   # this is the name of the key, whose value we're interested in. 
      ports:
        - containerPort: 3306
```

Now let's try it out:

```bash
$ mysql -h $(minikube ip) -P 31306 -u root -p'password123'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.15 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Success!

In this mysql example, everytime you delete the mysql pod, all data stored inside the mysql database get's deleted as well, which isn't good. That's why you should use Kubernetes Persistent Volumes for storing persistant data. Will cover more about Kubernetes Volumes later.


### Inject secrets as files

In this method, each key-name in a key/value secret pair, becomes the name of a text file, and the content of that file just contains the actual secret and nothing else. The yaml looks like this:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-centos
spec:
  volumes:                  # this approach requires the use of volumes, of the type 'secret'. 
    - name: my-secrets
      secret:
        secretName: mysecrets
  containers:
    - name: cntr-centos
      image: centos
      volumeMounts:
        - name: my-secrets
          mountPath: /etc/secrets      # this folder will house one or more files, one for each key/value secret pair. 
      command: ["/bin/bash", "-c"]
      args:
        - |
          while true ; do
            date
            sleep 10 
          done
```

Once the pod is created, we can see if the secrets are stored inside:


```bash
$ kubectl exec pod-centos -it /bin/bash -c cntr-centos
[root@pod-centos /]# cd /etc/secrets
[root@pod-centos secrets]# ls -l
total 0
lrwxrwxrwx 1 root root 20 Mar 12 13:22 password1.txt -> ..data/password1.txt
lrwxrwxrwx 1 root root 20 Mar 12 13:22 password2.txt -> ..data/password2.txt

[root@pod-centos secrets]# cat password1.txt
password123[root@pod-centos secrets]# cat password2.txt
passwordxyz[root@pod-centos secrets]# 
```

Note: the cat outputs looks a bit messy becuase the files content doesnt include any new-line charaacters, which is as it should be. 