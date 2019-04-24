# tls for pods

If you have web service pods, then you need to want to use https rather than http. First need to install cfssl:


## Install cfssl

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

## Create key and csr

Now let's create our private key and it's csr:


```bash
$ cat <<EOF | cfssl genkey - | cfssljson -bare server
> {
>   "hosts": [
>     "svc-clusterip-httpd.default.svc.cluster.local"
>   ],
>   "CN": "dep-httpd-*.default.svc.cluster.local",
>   "key": {
>     "algo": "ecdsa",
>     "size": 256
>   }
> }
> EOF
2019/04/17 20:52:34 [INFO] generate received request
2019/04/17 20:52:34 [INFO] received CSR
2019/04/17 20:52:34 [INFO] generating key: ecdsa-256
2019/04/17 20:52:34 [INFO] encoded CSR

$ ls -l | grep server
-rw-r--r--  1 root root  497 Apr 17 20:52 server.csr
-rw-------  1 root root  227 Apr 17 20:52 server-key.pem
```

Now let's create the kubernetes version of the csr file:

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: svc-clusterip-httpd.default
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
certificatesigningrequest.certificates.k8s.io/svc-clusterip-httpd.default created

$ kubectl get csr
NAME                          AGE   REQUESTOR          CONDITION
svc-clusterip-httpd.default   17s   kubernetes-admin   Pending
```

Next get this csr signed:

```bash
$ kubectl certificate approve svc-clusterip-httpd.default
certificatesigningrequest.certificates.k8s.io/svc-clusterip-httpd.default approved

$ kubectl get csr
NAME                          AGE     REQUESTOR          CONDITION
svc-clusterip-httpd.default   2m16s   kubernetes-admin   Approved,Issued
```

The resulting certificate is actually embeded into the csr object, and is base64 encoded, so to retreive it, we run:


```bash
$ kubectl get csr svc-clusterip-httpd.default -o jsonpath='{.status.certificate}' | base64 --decode > server.crt

$ ls -l server.crt
-rw-r--r-- 1 root root 960 Apr 17 20:58 server.crt
```

Now we are ready to feed in this server.crt and server-key.pem into our pods. we can create a configmap and generic certificate to do this, but a more elegant solution is to create a tls based k8s secret:


```bash
$ kubectl create secret tls httpd-tls --cert=server.crt --key=server-key.pem
secret/httpd-tls created

$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-95psk   kubernetes.io/service-account-token   3      22h
httpd-tls             kubernetes.io/tls                     2      35s

$ kubectl describe secrets httpd-tls
Name:         httpd-tls
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  960 bytes
tls.key:  227 bytes
```

Now we recreate our deployments to make use of these secrets:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-httpd
spec:
  replicas: 2 
  selector:
    matchLabels:
      component: httpd_webserver
  template:
    metadata:
      labels:
        component: httpd_webserver
    spec:
      volumes:
      - name: tls-files 
        secret:
          secretName: httpd-tls
      containers:
        - name: cntr-httpd
          image: httpd:latest
          volumeMounts:
            - name: tls-files
              mountPath: /usr/local/apache2/tls        # this folder will get created
              readOnly: true
          ports:
            - containerPort: 80
          command: ["/bin/bash", "-c"]
          args:
            - |
              # this sed command was taken from https://hub.docker.com/_/httpd
              sed -i \
                  -e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
                  -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
                  -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
                  conf/httpd.conf
              
              cp conf/extra/httpd-ssl.conf conf/extra/httpd-ssl.conf-orig
              sed -i \
                  -e 's:/usr/local/apache2/conf/server.crt:/usr/local/apache2/tls/tls.crt:' \
                  -e 's:/usr/local/apache2/conf/server.key:/usr/local/apache2/tls/tls.key:' \
                  conf/extra/httpd-ssl.conf
              
              /bin/echo "You've hit httpd - $HOSTNAME" > /usr/local/apache2/htdocs/index.html
              /usr/local/bin/httpd-foreground
```

The sed comamnds are needed to configure ssl in our containers. 

We also create our curl client pod and ensure that we add in the kubernetes ca's cert as a trusted root cert:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-curl-client
  labels:
    app: curl_client 
spec:
  containers:
    - name: cntr-centos
      image: centos
      command: ["/bin/bash", "-c"]
      args:
        - |
          # the following is to add the kubernete's root ca cert as a trusted cert. 
          # the root ca crt is already present inside the container thanks to the default service account. 
          cp /run/secrets/kubernetes.io/serviceaccount/ca.crt /etc/pki/ca-trust/source/anchors/
          update-ca-trust   # ubuntu approach: https://kubernetes.io/docs/concepts/cluster-administration/certificates/#distributing-self-signed-ca-certificate
          update-ca-trust extract
          while true ; do
            echo "################################"
            date
            curl -s https://svc-clusterip-httpd.default.svc.cluster.local
            sleep 10
          done
```


You can also access this service from your web browser, but you need to add in the ca.crt file to your google chrome browser as a trusted cert. In a company where everyone uses Microsoft windows company laptops/workstations, this task can be done centrally, using [group policy](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/distribute-certificates-to-client-computers-by-using-group-policy).

Also the fact that we created a secret of the type 'tls'. That's actually syntax sugar. You could have achieved the same task by just creating normal generic secrets to store your .crt and .key files. In fact it makes more sense to store the .crt file in a configmap instead, since it's not really a secret. 












## Reference
[https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) - to provide a secure private service

[https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/) - for provide a secure public service
