# backup and retore Kubernetes

Kubernetes doesn't use a database for storing data, instead it uses kev-value storage software called etcd to store the cluster's state. So it's important to know how to backup/restore your etcd storage. 





## Install and configure the etcdctl utility
Before you can do etcd backups/restore, you first need to install the binary and configure it.  To perform backups you need to use the etcdctl binaries and it needs to be the version mentioned in the [docs](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#prerequisites). In our case we'll use version 3.2.10. 

In ubuntu the easiest way to install packages is using apt-get. However in our case ubuntu doesn't have the newer version of etcdctl:

```bash
$ apt-get install apt-file
$ apt-file update
$ apt-file find etcdctl
etcd: /usr/bin/etcdctl
etcd: /usr/share/man/man1/etcdctl.1.gz
...
$ apt-cache madison etcd
      etcd | 2.2.5+dfsg-1ubuntu1 | http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
      etcd | 2.2.5+dfsg-1 | http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages
```




The etcd binaries isn't available in the kubernetes repo, so you need to download them from this url:


```bash
etcd_version=3.2.10
curl -L https://github.com/coreos/etcd/releases/download/v${etcd_version}/etcd-v${etcd_version}-linux-amd64.tar.gz -o etcd-v${etcd_version}-linux-amd64.tar.gz
tar -xvzf etcd-v${etcd_version}-linux-amd64.tar.gz
cd etcd-v${etcd_version}-linux-amd64
```

Then locate the binary and move it to one of your $PATH folders, then make it executable:


```bash
$ cp etcdctl /usr/local/bin/
$ etcdctl --version
etcdctl version: 3.2.10
API version: 2
```

There are 2 'API versions' built into this etcdtl:

```bash
# etcdctl --help
NAME:
   etcdctl - A simple command line client for etcd.

WARNING:
   Environment variable ETCDCTL_API is not set; defaults to etcdctl v2.
   Set environment variable ETCDCTL_API=3 to use v3 API or ETCDCTL_API=2 to use v2 API.
...
```

Kubernetes uses version 3, so we need to activate that version for our current shell:

```bash
echo 'export ETCDCTL_API=3' > /etc/profile.d/etcdctl.sh
export ETCDCTL_API=3
```

Then check again:

```bash
# etcdctl version
etcdctl version: 3.2.10
API version: 3.2
```

Notice that this time we omitted '--' in '--version', that's because this version of the binary has slightly different command syntaxes. 

## Using etcdctl

Now that we have setup etcdctl, we can now start using it. Everytime you interact with etcdctl you need to authenticate using tls certificates, similar to the way do when using kubectl. Here are the commands:

```bash
$ etcdctl member list --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
730157f1ac45b3cf, started, kube-master, https://10.2.5.110:2380, https://10.2.5.110:2379
```

Here we specified 3 certs. If this cluster is built with kubeadm then these certs are all under /etc/kubernetes/pki/etcd/. You always have to specify these 3 flags when using the etcdctl flag. 


## Create backups

Now that we have setup the etcdctl, we can now create a backup by running:


```bash
$ etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
Snapshot saved at snapshot.db

$ ll -h snapshot.db
-rw-r--r-- 1 root root 2.1M Apr 13 11:27 snapshot.db
```

You can then check if your snapshot.db is valid by running:

```bash
$ etcdctl --write-out=table snapshot status snapshot.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 1c0d2b9f |    14323 |       1051 |     2.2 MB |
+----------+----------+------------+------------+
```

Now you need to store this snapshot.db and all the `/etc/kubernetes/pki/etcd` folder in a places, e.g. AWS S3 buckets.

## Restore backups


covered later. 





## Reference

[https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)


[performing etcd restore](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md)