# Service endpoints

Earlier we saw how ClusterIP service objects are used:

- by other pods to communicate with eachother using a dns entry
- loadbalancing a cluster of pods


However what if you want your pods to access an external server which is accessible by an external IP address? You can feed in the external IP addreess into your pods via environment variables, but if the ip address then changes? Luckily you can address this problem using ClusterIP service objects again, but this time you omit specifying a selector section:

```yaml
$ kubectl describe service my-external-service | grep -i endpoint

```


At ths stage, this service has no idea where to forward any traffic to:


```bash
$ kubectl get service
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP   111s
my-external-service   ClusterIP   10.111.236.124   <none>        80/TCP    65s

$ kubectl describe service my-external-service | grep endpoint
$
```

To fix this, we create an Endpoint object:

```yaml
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-external-service    # this name must match the ClusterIP service name
subsets:
  - addresses:
    - ip: 77.104.171.177       # nslookup codingbee.net. 
#   - ip: x.x.x.x          # add more if available for loadbalancing purposes. 
    ports:
      - port: 443
```

Now if you log into a pod we should be able to run this:

```bash
$ kubectl exec -it pod-centos -c cntr-centos -- /bin/bash
[root@pod-centos /]# nslookup my-external-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   my-external-service.default.svc.cluster.local
Address: 10.100.23.177

[root@pod-centos /]# curl -k https://my-external-service
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<style type="text/css">
        body{background-color:#f4f9fa}
        .centered{text-align:center}
```

Here the pods is accessing this external service like any other internal service. In this case, the curl command fails, only becuase the external server server didn't like the name 'my-external-service'. We can circumvent this with:

```bash
curl -k https://my-external-service --header "Host: codingbee.net"
<!DOCTYPE html>
<html lang="en-US">
<head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
        <title>codingbee &#8211; &quot;Building Immutable Cloud Infrastructure one container at a time!&quot;</title>
<link rel='dns-prefetch' href='//s.w.org' />
...
```

To summarise, what we have effective created is an A-Name DNS entry (ip-address to DNS Alias) in Kubernetes's internal dns service (CoreDNS). 

However you can setup a C-Name DNS entry (DNS Alias to another DNS Alias). That's done by creating service objects of the type 'ExternalName'.