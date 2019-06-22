# StorageClass

Earlier we saw that in order for an App Developer to create a PVC, the KA needs to first manually create a suitable PV object beforehand. Manually creating the same type of PV over and over again can become tedious. That's why in Kubernetes you can automate this so that a PV is automically creates as an when a new PVC requests for it, i.e. dynamically provisions VM on-demand. That's done by making use of [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) objects.

Like PVs, StorageClass are cluster level objects. So objects in any namespace can make use of it. A PVC can submit a request to a StorageClass and the StorageClass can create a PV upon requests.

[StorageClasses doesn't support all PV types (e.g. NFS) yet](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner).

Also the type of storage class you can create depends on the underlying infrastructure your kube cluster running on, e.g. AWS, Azure,...etc.

## The Default Storage Class (eg1-default-StorageClass)

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

This is the default storageclass. This means that if a PVC is created, but there are no PVs that meets the PVC's requirements, then this storageclass (being the default) will step in and create the PV to meet the PVC's request. So to make use of this storage class simply create a PVC with no suitable preprovisioned PVs. In this demo we start with no PVs or PVCs:

```bash
$ kubectl get pvc
No resources found.
$ kubectl get pv
No resources found.
```

So when we apply the PVC:

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-db-data-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

We end up with:

```bash
$ kubectl get pvc
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-db-data-storage   Bound    pvc-8a36c914-4ef5-11e9-8cda-0800271f16b7   1Gi        RWO            standard       3s
```

Here we can see that this PVC's requests was dynamically provisioned, by the 'standard' StorageClass.

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
pvc-8a36c914-4ef5-11e9-8cda-0800271f16b7   1Gi        RWO            Delete           Bound    default/pvc-db-data-storage   standard                4m32s
```

Notice here that the reclaim policy is 'Delete'. This means if we delete the PVC, then the PV also get's deleted:

```bash
$ kubectl delete pvc pvc-db-data-storage
persistentvolumeclaim "pvc-db-data-storage" deleted
$ kubectl get pvc
No resources found.
```

## Custom StorageClasses

What if we want the reclaim policy to be 'Retain'. That setting is specified inside the storage class object, so to change this you can edit the 'standard' storage class, but that will impact all future PVCs. So a better approach would be create a new custom storage class (you can use the existing standard sc as a template to save time). Here's the custom storage class I'm going to use:

```yaml
---
# I created this by using 'kubectl edit sc standard' as a template
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "false"
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
  name: standard-persist
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Retain
volumeBindingMode: Immediate

```

After applying this we get:

```bash
$ kubectl get sc
NAME                 PROVISIONER                AGE
standard (default)   k8s.io/minikube-hostpath   51m
standard-persist     k8s.io/minikube-hostpath   4s
```

Now in our PVC yaml file, we need to specifically request for this storageclass,`pvc.spec.storageClassName`, to avoid it from using the default.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-db-data-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard-persist     # Here we specify the storageclass to use, if no suitable preprovisioned PVs are available
```

Now when we apply this we get:

```bash
$ kubectl get pvc
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
pvc-db-data-storage   Bound    pvc-5a3887c3-4efc-11e9-8cda-0800271f16b7   1Gi        RWO            standard-persist   6m46s
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS       REASON   AGE
pvc-5a3887c3-4efc-11e9-8cda-0800271f16b7   1Gi        RWO            Delete           Bound    default/pvc-db-data-storage   standard-persist            6m48s
```

Looks like a [bug](https://github.com/kubernetes/minikube/issues/3955).

## Disabling Dynamic Provisioning

If you want your PVC to only use pre-provisioned PVs, and not use the default storage class, then you do that by setting storageClassName to an empty string:

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-db-data-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ''    # empty string.
```

In this scenario, this PVC will continuously show as pending if no matching pre-provisioned PV is found.

```bash
$ kubectl get pvc
NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-db-data-storage   Pending                                                     100s

$ kubectl get pv
No resources found.
```
