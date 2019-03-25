# StorageClass

Earlier we saw that in order for an App Developer to create a PVC, the KA needs to first create a PV object manually. Manually creating the same type of PV over and over again can become tedious. That's why in Kubernetes you can dynamically provisions VM on-demand by making use of [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) objects.

Like PVs, StorageClass objects exists at the kube cluster level as opposed to the namespace level. So objects in any namespace can make use of it. A PVC can submit a request to a StorageClass and the StorageClass can create a PV upon requests.

[StorageClasses doesn't support all PV types (e.g. NFS) yet](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner).

Also the type of storage class you can create depends on the underlying infrastructure your kube cluster running on, e.g. AWS, Azure,...etc.

One cool feature is that during a kubernetes installation, Kubernetes identifies what infrastructure it is being installed on and automatically create a default StorageClass object, e.g. Minikube creates a storageclass using Minikube's own bespoke [provisioner](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner), which is based on hostPath PV type.


```bash
$ kubectl get storageclass
NAME                 PROVISIONER                AGE
standard (default)   k8s.io/minikube-hostpath   9d


$ kubectl describe storageclass standard
Name:                  standard
IsDefaultClass:        Yes
Annotations:           storageclass.beta.kubernetes.io/is-default-class=true
Provisioner:           k8s.io/minikube-hostpath
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

This is the default storageclass. This means that if a PVC is created, but there are no PVs that meets the PVC's requirements, then this storageclass (being the default) will step in and create the PV to meet the PVC's request.

```bash


```









