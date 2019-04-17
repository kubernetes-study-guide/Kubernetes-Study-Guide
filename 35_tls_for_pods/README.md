# tls for pods

If you have web service pods, then you need to want to use https rather than http. First need to install cfssl:


## Install cfssl
```bash
curl -SL https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -o /usr/local/bin/cfssl
chmod ugo+x /usr/local/bin/cfssl

curl -SL https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -o /usr/local/bin/cfssljson
chmod ugo+x /usr/local/bin/cfssljson
```

These [urls are documented](https://kubernetes.io/docs/concepts/cluster-administration/certificates/), which you can find by searching for cfssl in kubernetes website. It's the first match. 

Next check that they have installed correctly:

```bash
$ cfssl version
Version: 1.2.0
Revision: dev
Runtime: go1.6

$ cfssljson -help
Usage of cfssljson:
  -bare
    	the response from CFSSL is not wrapped in the API standard response
  -f string
    	JSON input (default "-")
  -stdout
    	output the response instead of saving to a file
```

# Create key and csr

Now let's create our private key and it's csr:


```bash
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "svc-clusterip-httpd.default.svc.cluster.local",
    "10.107.217.243"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```

Now let's create the kubernetes version of the csr file:

```bash
# cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: my-svc.my-namespace
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

# kubectl get csr
NAME                  AGE     REQUESTOR          CONDITION
my-svc.my-namespace   9m23s   kubernetes-admin   Approved,Issued
```

Next get this csr signed:

```bash
# kubectl certificate approve my-svc.my-namespace
certificatesigningrequest.certificates.k8s.io/my-svc.my-namespace approved
```

The resulting certificate is actually embeded into the csr object, and is base64 encoded, so to retreive it, we run:


```bash
$ kubectl get csr my-svc.my-namespace -o jsonpath='{.status.certificate}' | base64 --decode > server.crt
```

Now we are ready to embed this server.crt and server-key.pem into our pods. we can create a configmap and generic certificate to do this, but a more elegant solution is to create a tls certificate:


```bash
$ kubectl create secret tls httpd-tls --cert=server.crt --key=server-key.pem
secret/httpd-tls created

$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-95psk   kubernetes.io/service-account-token   3      22h
httpd-tls             kubernetes.io/tls                     2      35s
```




```











## Reference
[https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) - to provide a secure private service

[https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/) - for provide a secure public service