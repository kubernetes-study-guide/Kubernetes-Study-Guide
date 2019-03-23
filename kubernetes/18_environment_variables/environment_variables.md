# Environment Variables

Some Docker images, e.g. the [official mysql image](https://hub.docker.com/_/mysql) let's you feed in [docker image environment variables](https://hub.docker.com/_/mysql#environment-variables) into the the container. These environment variables are usually optional, but some can be mandatory. In the case of the mysql image, the MYSQL_ROOT_PASSWORD variable is mandatory. These environment variables are usually used by an [entrypoint](https://github.com/docker-library/mysql/blob/master/8.0/docker-entrypoint.sh) script during a container's launch time.

Here we're going to look at how we feed in environment variables into a container using kubernetes. We'll use the official mysql image for this demo.

The [mysql docker image environment variables](https://hub.docker.com/_/mysql#environment-variables) documentation tells us what env variables are available, so we now construct a pod definition yaml file with the content:

```yaml
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
      env:                       # we use 'env' section to feed in environments variables
        - name: MYSQL_DATABASE      # this is an optional environment variable
          value: dummy_db
        - name: "MYSQL_ROOT_PASSWORD"  # this is a mandatory environment variable
          value: "password123"
      ports:
        - containerPort: 3306
```

Note: We have defined MYSQL_ROOT_PASSWORD in plain text above. That's not best practice, we'll cover a better approach using 'secret', covered later. Environment variables must be in the form of a string. Hence any variable that is a number, must be enclosed in single quotes. Now lets build this:

```bash
$ kubectl apply -f configs
pod/pod-mysql-db created
service/svc-nodeport-mysql-db-server created
```

This seems to have worked:

```bash
$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pod-mysql-db   1/1     Running   0          39s

$ kubectl get svc
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                     ClusterIP   10.96.0.1       <none>        443/TCP          4m46s
svc-nodeport-mysql-db-server   NodePort    10.111.211.39   <none>        3050:31306/TCP   43s
```

Let's now check if our env variables exist inside the mysql container:

```bash
$ kubectl exec -it pod-mysql-db /bin/bash
root@pod-mysql-db:/# env | grep PASSWORD
MYSQL_ROOT_PASSWORD=password123
root@pod-mysql-db:/# env | grep MYSQL_DATABASE
MYSQL_DATABASE=dummy_db
```

So it looks likes this has worked and it means that these environment variables should be available to the [Docker image's entrypoint script](https://github.com/docker-library/mysql/blob/master/8.0/docker-entrypoint.sh).

But the only way to know for sure is to take a look inside the pod by creating a mysql session inside it.

## Creating an interactive mysql session to a pod

Let's start by finding out what ip address we should be using:

```bash
$ minikube ip
192.168.99.102
```

Now let's see if we can nc/telnet to it (using the port number that's listed in our svc object as shown above):

```bash
$ nc -v 192.168.99.102 31306
found 0 associations
found 1 connections:
     1: flags=82<CONNECTED,PREFERRED>
        outif vboxnet7
        src 192.168.99.1 port 54368
        dst 192.168.99.102 port 31306
        rank info not available
        TCP aux info available

Connection to 192.168.99.102 port 31306 [tcp/*] succeeded!
```

This means our service object is working, and our pod is listening on port 3306. Next we'll install mysql client, on our macbook (if we don't already have it installed):

```bash
brew install mysql-client
```

Now lets try establishing a mysql connection and see if our dummy_db exists:

```bash
$ mysql -h 192.168.99.102 -P 31306 -u root -p'password123'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.15 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| dummy_db           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql>
```

Success! however note that dummy_db was created because of the env variables we fed in via the yaml configurations. if dummy_db contained data, then that data would get wiped out if you rebuild the pod, leaving you with a empty dummy_db db again. To make the db and it's data persistant, we need to make use of persistant volumes, covered later.

Now let's delete everything:

```bash
$ kubectl delete -f configs
pod "pod-mysql-db" deleted
service "svc-nodeport-mysql-db-server" deleted
```

## Inject Pod metadata into containers

You can also environment variables to [inject pod metadata into containers](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/).