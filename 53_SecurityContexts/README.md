# security contexts







```bash
man capabilities
```


## Node level capabilities

The following makes changes at worker node level:

- `pod.spec.hostNetwork`
- `pod.spec.containers.ports.hostPort`
- `pod.spec.hostPID`
- `pod.spec.hostIPC`

These kind of features are typically used by pods in the kube-system namespace, and causes behaviour that makes it appear that the pod's application are installed directly on the worker nodes. 