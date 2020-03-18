need to perform install of kubeadm, docker, 

configure sysctl

https://github.com/weaveworks/weave/issues/3363#issuecomment-408767191

Had to run route traffic 10.96.0.1/32 to eth1:

```
ip route add 10.96.0.1/32 dev eth1
```

Then confirmed new rout has been added:

```
ip route get 10.96.0.1
```

Then run the kubectl init command. 