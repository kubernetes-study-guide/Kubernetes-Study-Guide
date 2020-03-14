# ETCD


```bash
etcd_version=3.2.10
curl -L https://github.com/coreos/etcd/releases/download/v${etcd_version}/etcd-v${etcd_version}-linux-amd64.tar.gz -o etcd-v${etcd_version}-linux-amd64.tar.gz
tar -xvzf etcd-v${etcd_version}-linux-amd64.tar.gz
cd etcd-v${etcd_version}-linux-amd64
```

The tar archive contains 2 binaries:

-  `etcd` - This is the etcd server-side binary that runs the etcd service
-  `etcdctl` - This is the client-side binary that's used for interacting with the etcd service. 