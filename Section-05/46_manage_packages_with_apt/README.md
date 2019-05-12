# Using apt and dpkg

This article isn't about Kubernetes. It's how to use ubuntu/debian native package manager for installing packages from the official repos. This is a crash course if you are new to apt.

Update local apt repo index (run this command regularly to stay up to date):

```bash
apt-get update
```

It makes use of files in `/etc/apt/sources.list.d/`. Here's an example:

```bash
# cat /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
```

[List all installed packages](https://askubuntu.com/a/17829/848501):

```bash
apt list --installed
```

[List all available packages](https://askubuntu.com/a/160899/848501):

```bash

apt-cache search .

```

Install a package:

```bash
apt-get install --only-upgrade kubeadm
```

[Find which package provides file](https://askubuntu.com/a/1912/848501) called 'etcdctl':

```bash
apt-get install apt-file
apt-file update
apt-file find etcdctl
```

The above is a wildcard search, here's a more specific search:

```bash
$ apt-file find /usr/bin/etcdctl
etcd: /usr/bin/etcdctl
```

List all versions of a particular package:

```bash
$ apt-cache madison kubeadm
   kubeadm |  1.14.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
...

```

Install a particular version of a package:

```bash
apt-get install kubeadm=1.14.0-00
```

Notice that we used the "=" sign for specify a version.

If you want to change a version of an installed package to a particular version:

```bash
apt-get install --only-upgrade kubeadm=1.14.0-00
```

[List all files in a package that is currently installed](https://askubuntu.com/a/32509/848501):

```bash
dpkg-query -L kubeadm

```

[List all files that would be installed by installing a package]:

```bash
apt-get install apt-file
apt-file update
apt-file list etcd
```

Get info about a particular package:

```bash
apt-cache show kubeadm
```