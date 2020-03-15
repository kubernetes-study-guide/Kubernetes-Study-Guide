# Persistant Volume Claims

> I've created a [kubernetes-NFS-Persistant-Volume-Vagrant](https://github.com/Sher-Chowdhury/kubernetes-NFS-Persistant-Volume-Vagrant-Demo) for this article so that you can follow along this demo.

So far we've covered how an Application App Developer can use persistent volumes by adding the necessary section under their pod spec. However, there is a problem with this approach, the developers (who are packaging their apps into docker-images and pods are) likely to have only a basic understanding of the kubernete cluster's underlying cloud/on-premise infrastructure. In fact as developers, it's not in their remit to have that kind of knowledge because Kubernetes is supposed to allow developers to develop without having to worry about whether their app is running in AWS, Azure,....etc.

So filling in things like NFS server ip address in their yaml files becomes an undesired burden on the developers. A better solution would be for the Kubernetes Administrators to create the PV's as standalone Kubernetes Objects that are then available for the developers to reference in their pod yaml definitions. That's possible, thanks to **Persistent Volume Claims (aka PVCs)**. PVC can

 1. use existing PVs if there is a suitable PV that meets the requirements.  
 2. dynamically provisions PVs

In both scenarios, the application developer doesn't need to create PVs. instead they just create PVC objects instea.

## Use existing PVs

Let's take a look at both these approaches using AWS's AWSElasticBlockStore as an example. Following on from the previous NFS example, the Kubernetes Administrator (KA) is the person who not only builds+maintains the kubecluster, but also manages the underlying infrastructure, e.g. AWS, Azure, On-Premise,...etc. In this scenario, the KA could also be the person who built the NFS server too. Once the NFS server is available, the KA would then create the PV:

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-db-data-storage
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce    # aka RWO
    - ReadOnlyMany     # aka ROX
    - ReadWriteMany    # aka RWX
  persistentVolumeReclaimPolicy: retain
  nfs:
    path: /nfs/export_rw
    server: 10.3.5.109
```

There are a few features to highlight abou this yaml file:

- **[accessModes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)**: The various ways available to a worker node (not pods). They are mutually exclusive, so you can only opt for one of available options during mount time.
  - **ReadWriteOnce** – the volume can be mounted as read-write by a single node
  - **ReadOnlyMany** – the volume can be mounted read-only by many nodes
  - **ReadWriteMany** – the volume can be mounted as read-write by many nodes
- **[persistentVolumeReclaimPolicy](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy)**: This defines what to do with a PV's content want's it getes released by a PVC.
  - **Retain** – manual reclamation. The PV's data is retained after it's associated PVC is deleted. However the PV status becomes 'pending' rather than available. That's to give the KA a chance to (if necessary) to backup the PV's content and then make it available, be recreating the PV.
  - **Recycle** – Delete all data stored inside the PV, so that it can be used as a clean PV.
  - **Delete** – Delete the whole PV

The above yaml ends up creating:

```bash
$ kubectl get persistentvolume
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
db-data-storage   1Gi        RWO            Retain           Available                                   38m


$ kubectl describe persistentvolume
Name:            db-data-storage
Labels:          <none>
Annotations:     kubectl.kubernetes.io/last-applied-configuration:
                   {"apiVersion":"v1","kind":"PersistentVolume","metadata":{"annotations":{},"name":"db-data-storage"},"spec":{"accessModes":["ReadWriteOnce"...
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    10.3.5.109
    Path:      /nfs/export_rw
    ReadOnly:  false
Events:        <none>

```

Persistant Volumes are actually kube cluster level objects, i.e. they don't belong to any namespaces. That means that objects in any namespaces can make use of it. After creating the PV, the KA would then notify the Developer that the PV is now ready. The Developer then 'claims' the PV by creating a PVC:

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
  volumeName: pv-db-data-storage
```

Notice, how much easier this is to write for the App Developer, there's no need specify to specify any technology related settings, e.g. nfs IP addresses.

As soon as you apply the above yaml, we don't explicitly refer to a particular PV object, instead it specifies some of the storage requirements. Kubernetes will therefore cross-reference this PVC's requirements against all available PVs to find a suitable match. In particular the PV needs to:

- Support the accessMode requested by the PVC
- Offer equal to or more storage requested by the PVC
- If 'volumeName' is used then pv name needs to match too. So may be convenient to omit this.

In our case our PV meeds the needs of the PVC so the PVC successfully 'bounds' itself to the PVC, i.e. the PVC successfully claimed the PV:

```bash
$ kubectl get pvc
NAME                  STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-db-data-storage   Bound    pv-db-data-storage   1Gi        RWO                           68s


$ kubectl describe pvc pvc-db-data-storage
Name:          pvc-db-data-storage
Namespace:     default
StorageClass:
Status:        Bound
Volume:        pv-db-data-storage
Labels:        <none>
Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                 {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"pvc-db-data-storage","namespace":"default"},"spec":...
               pv.kubernetes.io/bind-completed: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Events:        <none>
Mounted By:    pod-mysql-db
#
```

The status here shows 'Bound' which means the the developer's pvc has successfully claimed the PV. Now the developer can use the PV by referencing the PVC in the pod spec:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-mysql-db
  labels:
    component: mysql_db
spec:
  volumes:
    - name: db-data-storage
      persistentVolumeClaim:
        claimName: pvc-db-data-storage
  containers:
    - name: cntr-mysql-db
      image: mysql
      env:
        - name: "MYSQL_ROOT_PASSWORD"
          value: "password123"
      volumeMounts:
        - name: db-data-storage              # Here we call the PersistentVolume by it's name.
          mountPath: /var/lib/mysql
      ports:
        - containerPort: 3306
```

Now let's check if that has worked:

```bash
# kubectl exec -it pod-mysql-db -- /bin/bash
root@pod-mysql-db:/# df -h
Filesystem                    Size  Used Avail Use% Mounted on
overlay                        62G  2.6G   56G   5% /
tmpfs                          64M     0   64M   0% /dev
tmpfs                         497M     0  497M   0% /sys/fs/cgroup
/dev/mapper/vagrant--vg-root   62G  2.6G   56G   5% /etc/hosts
shm                            64M     0   64M   0% /dev/shm
10.3.5.109:/nfs/export_rw      41G  1.2G   40G   3% /var/lib/mysql
tmpfs                         497M   12K  497M   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                         497M     0  497M   0% /proc/acpi
tmpfs                         497M     0  497M   0% /proc/scsi
tmpfs                         497M     0  497M   0% /sys/firmware
```

## Dynamically provisions PVs

This is covered when we cover StorageClasses.

## IGNORE THE REST OF THIS ARTICLE

--------------------------------------

Persistant volumes are volumes that stores a container's data outside of a pod. So that when a pod dies, then the data still persists and get's used by a replacement container. hostpath volumes is an example of a persistant volume. However it has some limitations, in that:

- hostPath volumes are only accessible by pods that are on the same worker node.
- If worker nodes gets terminated then all hostPath volumes gets deleted.

 There are lots of other [Persistant Volume Plugins](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) available, some of which are cloud platform specific.

 There's a few ways to create PersistentVolumes. The first way is to define as part of the pod definition:

 ```yaml
 ---
apiVersion: v1
kind: Pod
metadata:
  name: pod-mysql-db
  labels:
    component: mysql_db
spec:
  volumes:
    - name: db-data-storage
      nfs:                      # here we are using nfs specification for the Persistent Volume
        path: /nfs/export_rw
        server: 10.3.5.109
        readOnly: false
  containers:
    - name: cntr-mysql-db
      image: mysql
      env:
        - name: "MYSQL_ROOT_PASSWORD"
          value: "password123"
      volumeMounts:
        - name: db-data-storage              # Here we call the PersistentVolume by it's name.
          mountPath: /var/lib/mysql
      ports:
        - containerPort: 3306
 ```

 -statically
 -dynamically (by using persistantVolumeClaims)
  Let's take a look at both these approaches using AWS's AWSElasticBlockStore as an example.

## Statically Provisioned Persistant Volumes

In this approach we manually create an EBS volume:

```bash
aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2
```

This volume needs to be in the same AWS AZ as the worker node that it will be attached to. Then you use the ebs volume like this:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    component: apache_webserver
spec:
  volumes:
    - name: webcontent
      awsElasticBlockStore:
        volumeID: <volume-id>             # need to hardcode the volume id in.
        fsType: ext4
  containers:
    - name: cntr-httpd
      image: httpd
      volumeMounts:
        - name: webcontent
          mountPath: /usr/local/apache2/htdocs
      ports:
        - containerPort: 80
```

This approach has a few pros and cons, depending on individual's permissions/roles/responsibilities in a workplace. For example a pod creator may need to seek company approval to get an EBS block created (since they cost money). Once their request is approved, the request could then be passed on to an AWS administrator to be actioned. The AWS administrator would then inform the pod creator of the volume id to use.

However in other workplaces, the pod ceator may have full rights+approvals to create EBS volumes. In which case, manually creating ebs then copy+pasting volume id into the yaml file is not an elegant solution. Instead it may be better to dynamically provision it.

## Dynamically Provision Persistant Volumes

In this approach, you can get kubernetes to create an EBS volume for you on-demand, i.e. [dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). This involves creating 3 types of Kubernetes objects:

- [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) object - this object is specifically used by PVC to help create PV. If you manually create the PV, then you can attach the PV to the PVC and skip needing the StorageClass.
- [Persistent Volume Claim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) object
- Persistant Volume object

PVC has a 1-2-1 mapping to a PV.

Here's the steps involved:

1. pod object - Your pod definition defines a volume it wishes to mount. The volume is referenced with a PVC.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-mysql-db
  labels:
    component: mysql_db
spec:
  volumes:
    - name: db-data-storage
      persistentVolumeClaim:
        claimName: pvc-mysql-db   # here we specify which PVC should request for this pod's PV
  containers:
    - name: cntr-mysql-db
      image: mysql
      env:
        - name: "MYSQL_ROOT_PASSWORD"
          value: "password123"
      volumeMounts:
        - name: db-data-storage
          mountPath: /var/lib/mysql
      ports:
        - containerPort: 3306
```

2.PVC object - Sends a request to the StorageClass object for a new PV. This essentially means that a PVC is a controller object. The PVC references the storage class.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-mysql-db
spec:
  accessModes:
    - ReadWriteOnce   # this means that corresponding PV can only be used by a single node. Also see: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
  resources:
    requests:
      storage: 100Mi
      storageClassName: standard  # This specifies which the StorageClass's name. It defaults to 'standard' if you omit this line
```

3.StorageClass - This in effect is a controller object. This creates the EBS volume as per the PVC's request, and publishes in the form of a persistent volume.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
  creationTimestamp: "2019-03-14T19:50:50Z"
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
  name: standard
  resourceVersion: "345"
  selfLink: /apis/storage.k8s.io/v1/storageclasses/standard
  uid: 77e456f6-4692-11e9-b5c9-08002722e1d0
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

The 'standard' storageClass object actually comes included with a kubernetes installation.

```bash
$ kubectl get storageclass
NAME                 TYPE
standard (default)   k8s.io/minikube-hostpath


$ kubectl describe storageclass standard
Name:           standard
IsDefaultClass: Yes
Annotations:    storageclass.beta.kubernetes.io/is-default-class=true
Provisioner:    k8s.io/minikube-hostpath
Parameters:     <none>
Events:         <none>
```

The Kubernetes install is smart enough to know which platform it is running on and will create a default 'standard' StorageClass object that's native to that platform. In our case our platform is minikube, so the storageclass.provisioner is minikube-hostpath shown above. We managed to get the above yaml content by running `kubectl edit storageclasses standard`.

When we apply the above yamls, we get:

```bash
$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-mysql-db   Bound    pvc-c160899f-4739-11e9-b5c9-08002722e1d0   100Mi      RWO            standard       16m
```

As mentioned earlier, pvc objects are controller objects, so the underying pv has also been created:

```bash
$ kubectl get pv -o wide
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-c160899f-4739-11e9-b5c9-08002722e1d0   100Mi      RWO            Delete           Bound    default/pvc-mysql-db   standard                16m
```

This PV is then attached to the pod:

```bash
$ kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-mysql-db   1/1     Running   0          15m   172.17.0.7   minikube   <none>           <none>


$ kubectl describe pod pod-mysql-db
Name:               pod-mysql-db
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Fri, 15 Mar 2019 15:51:21 +0000
Labels:             component=mysql_db
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"component":"mysql_db"},"name":"pod-mysql-db","namespace":"default"...
Status:             Running
IP:                 172.17.0.7
Containers:
  cntr-mysql-db:
    Container ID:   docker://8b374d4d178cb7d4b374a440c5ca863834d7ec981dc3860f8dfc9ecfc753e56e
    Image:          mysql
    Image ID:       docker-pullable://mysql@sha256:4589ba2850b93d103e60011fe528fc56230516c1efb4d3494c33ff499505356f
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 15 Mar 2019 15:51:24 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  password123
    Mounts:
      /var/lib/mysql from db-data-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ghr5j (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  db-data-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pvc-mysql-db
    ReadOnly:   false
  default-token-ghr5j:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ghr5j
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20m   default-scheduler  Successfully assigned default/pod-mysql-db to minikube
  Normal  Pulling    20m   kubelet, minikube  pulling image "mysql"
  Normal  Pulled     20m   kubelet, minikube  Successfully pulled image "mysql"
  Normal  Created    20m   kubelet, minikube  Created container
  Normal  Started    20m   kubelet, minikube  Started container
```

## Testing our Persistant Volume

On the surface it looks like our mysql data will now be persistant. Let's now try to test this. We'll test this by creating a mysql session, create a new dummy db, then exit out, rebuild the pod, then see if that dummy db still exists.

Now lets test this, first we install mysql client on our macbook:

```bash
brew install mysql-client
```

Now check if we can reach that port using nc or telnet:

```bash
$ minikube service svc-nodeport-mysql --url
http://192.168.99.106:31306



$ nc -v 192.168.99.106 31306
found 0 associations
found 1 connections:
     1: flags=82<CONNECTED,PREFERRED>
        outif vboxnet7
        src 192.168.99.1 port 56396
        dst 192.168.99.106 port 31306
        rank info not available
        TCP aux info available

Connection to 192.168.99.106 port 31306 [tcp/*] succeeded!
```

Now try establishing a mysql connection:

```bash
$ mysql -h 192.168.99.106 -P 31306 -u root -p'password123'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.15 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Now lets create a dummy db:

```bash
mysql> CREATE DATABASE dummy_db;
Query OK, 1 row affected (0.02 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| dummy_db           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit
Bye
```

Now let's rebuild our pod, and then see if our dummy db still exists:

```bash
$ mysql -h 192.168.99.106 -P 31306 -u root -p'password123'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.15 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| dummy_db           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>
```
