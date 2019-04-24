# Create new user

Let's say a new employee, Lisa, joined the company and you want to give them [kubectl access](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/). There's 2 stages involved in giving Lisa access:

1. Authentication
2. Authorisation

In this demo we'll work with the [kubeadm provisioned kubecluster](https://github.com/Sher-Chowdhury/kubernetes-the-kubeadm-way-vagrant), because minikube isn't setup to do this kind of demo.  


## Authentication

Ultimately, this stage involves creating a valid kubectl file, along with it's dependencies. 

Let's say you want to give Lisa access using the [client certs approach](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#x509-client-certs). 


**Step 1:** Lisa creates a private key and csr (or just create the csr if a Lisa already has a private key that she wants to use):

```bash
# Lisa create a folder to store her tls files
$ mkdir -p ~/.kube/tls
$ cd .kube/tls/

# Lisa create the private key, she has called it lisa.pem
$ openssl genpkey -algorithm RSA -out lisa.pem -pkeyopt rsa_keygen_bits:2048
.+++
..................................................................................................................................................................+++
$ ls -l
total 4
-rw-rw-r-- 1 lisa lisa 1704 Apr  7 13:45 lisa.pem

# Lisa creates the csr file
$ openssl req -new -key lisa.pem -out lisa.csr -subj "/CN=lisa/O=restricted/O=employee"
$ ls -l lisa.csr
-rw-rw-r-- 1 lisa lisa 940 Apr  7 13:48 lisa.csr
```

Here, the CN name is the username that will be used. The 2 'O' values relates to access permissions (authorisations) which will cover later. The O values are things that Dave would have advised to Lisa.  


Lisa then emails the csr file to Dave.

**Step 2:** Dave signs the CSR with the Kubecluster's CA file, and sends back the resulting crt back to lisa. 


```bash
# Dave first review the csr before signing. He wants to make sure the Subject section is correct. 
$ openssl req -in lisa.csr -noout -text | grep Subject | grep CN
        Subject: CN=lisa, O=restricted, O=employee

# Dave signs the csr, i.e. creates a crt file from the csr. 
$ openssl x509 -req -in lisa.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out lisa.crt
```
The locations of ca.crt and ca.key, are the default location for a kubeadm provisioned kubecluster. 

Dave then emails the certificate (lisa.crt) file back to lisa. 

Dave also emails the Kubecluster's Certificate Authority's certificate, `/etc/kubernetes/pki/ca.crt` to Lisa. She needs this to create her kubectl config file. 

Dave also emails the following output to Lisa:

```bash
# kubectl cluster-info 
Kubernetes master is running at https://10.2.5.110:6443
KubeDNS is running at https://10.2.5.110:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Note: the ip-address/port-number could be different if there's a loadbalancer sitting in front of the kubemasters.

**Step 3**: Lisa now creates kubectl config file. She now has everything she needs:

She first checks that the certificate is hers:

```bash
$ openssl x509 -in lisa.crt -text -noout | grep Subject | grep CN
        Subject: CN=lisa, O=restricted, O=employee
```

Then she creates her kubectl's `~/.kube/config` file: 

```bash
# Create the cluster, Lisa chose to call her cluster 'codingbee-cluster'
kubectl config set-cluster codingbee-cluster --server=https://10.2.5.110:6443 --certificate-authority=/home/lisa/kube-tls/ca.crt

# Create the credentials - she chose the name codingbee-lisa
kubectl config set-credentials codingbee-lisa --client-key=/home/lisa/kube-tls/lisa.pem --client-certificate=/home/lisa/kube-tls/lisa.crt

# Create the context
kubectl config set-context codingbee-default-lisa --cluster=codingbee-cluster --user=codingbee-lisa --namespace=default


# Activate the context
kubectl config use-context codingbee-default-lisa
```

At this point, when lisa now tries to use kubectl command:


```bash
$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "lisa" cannot list resource "pods" in API group "" in the namespace "default"
```

This shows that she made it past [authentication, but failed at the authorisation stage](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/). I.e. she is a valid user, but doesn't have any permissions yet. This is something Dave needs to sort out by setting up Roles-Based-Access-Control (RBAC) related resources.












## Reference

[https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/)
