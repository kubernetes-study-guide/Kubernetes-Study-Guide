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

Now let's 










## Reference
[https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) - to provide a secure private service

[https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/) - for provide a secure public service