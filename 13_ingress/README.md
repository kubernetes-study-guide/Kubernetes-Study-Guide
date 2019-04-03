# Ingress

Earlier we covered service objects which sets up networking between pods in the same kubecluster, by creating a ClusterIP service, and also how we can make a pod externally accessible by creating a NodePort service. However as mentioned, Nodeport shouldn't be used in a production environment. 

The ideal solution would be to have your worker nodes only listening to standard ports, e.g. port 443 for https. That's possible by setting up Ingress objects. 

Ingress ojects are used to make pods externally accessible. Before you can start creating ingress objects, you first need to setup an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). An ingress controllers provides Forward Proxy, 'Edge router' and 'Loadbalancing' features. One popular choice is [Traefik](https://github.com/containous/traefik). However in our case we'll use [ingress-nginx github repo](https://github.com/kubernetes/ingress-nginx), which essentially makes use of an internal nginx forward-proxy pod behind the scenes. 

The way you set up the ingres-nginx controller depends on what underlying platfrom you're using, in our examples we'll be using minikube. 


## Setting up Ingress Controller on Minikube

The [Nginx official Ingress](https://kubernetes.github.io/ingress-nginx/) Documentation covers how to set up the Ingress object. First go to the [Ingress deploy](https://kubernetes.github.io/ingress-nginx/deploy/) section. Then perform the [generic deployloyment](https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command), this part is irrespective of what cloud platform you're using:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created
```

This ends up creating a new namespace and creates objects inside that namespace.

```bash
$ kubectl get all --namespace=ingress-nginx
NAME                                            READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-797b884cbc-qddzz   1/1     Running   0          8m7s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-controller   1/1     1            1           8m7s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-controller-797b884cbc   1         1         1       8m7s
$ kubectl get configmap --namespace=ingress-nginx
NAME                              DATA   AGE
ingress-controller-leader-nginx   0      7m49s
nginx-configuration               0      8m35s
tcp-services                      0      8m35s
udp-services                      0      8m35s
```

Next we following the instructions to [enable ingress for minkube](https://kubernetes.github.io/ingress-nginx/deploy/#minikube):

```bash
$ minikube addons enable ingress
✅  ingress was successfully enabled
```











Now we create our test environment.

```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:      # this is something specific to ingress objects. It lets you customise your ingress setup.
    kubernetes.io/ingress.class: nginx  # this is how we tell k8s to build ingress controller using the nginx project.
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  rules:
    - http:       # this enables listening on port 80, i.e. http port
        paths:
          - path: /
            backend:
              serviceName: svc-clusterip-httpd
              servicePort: 4000
```

There are a lot of [Ingress Controller Annotation settings](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations) available that allows you to customise your Kubernetes setup. 

Applying this yaml results in:

```bash
$ kubectl apply -f configs/ingress-example/ingress-obj-def.yaml 
ingress.extensions/ingress-service created

$ kubectl get ingress
NAME              HOSTS   ADDRESS   PORTS   AGE
ingress-service   *                 80      4s
```

Now we can test it by running:

```bash
$ minikube ip
192.168.99.102

$ curl http://192.168.99.102
<html><body><h1>It works!</h1></body></html>
```

The important thing here is that we no longer need to specify a port number in the url. 


Noticed that we disabled some ssl setting annotations in the yaml file. That's just to avoid getting unwanted redirect messages, or insecure ssl certs message, which can still be suppressed using -Lk curl flags as shown below: 


```bash
$ curl http://192.168.99.102
<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx/1.15.6</center>
</body>
</html>


$ curl -L http://192.168.99.102
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isnt adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
HTTPS-proxy has similar options --proxy-cacert and --proxy-insecure.


$ curl -Lk  http://192.168.99.102
<html><body><h1>It works!</h1></body></html>

```



## Ingress Objects Demo

In this demo, we're going to start bringing together a lot of the topics covered earlier. In this demo we're going to use deployment objects to create 2 groups of pods, the 1st group will a bunch of apache pods, the second will be a group of Caddy pods. Caddy is a webserver software that's similar to Apache/httpd, but written Golang. The Apache pods will be listening on port 80, and the caddy pods by default will listen on pod 2015. Each group will have it's own ClusterIP object sitting in front of it. We'll then create an Ingress object that sit's in front of both ClusterIP objects and it will forward incoming external requests to the ClusterIP objects. The Ingress object will decide which ClusterIP object it will forward traffic to based on the requesting url address.


So let's start by creating our 2 deployments:

```bash
$ kubectl apply -f configs/eg1-ingress/dep-httpd.yml
deployment.apps/dep-httpd created
$ kubectl apply -f configs/eg1-ingress/dep-nginx.yml
deployment.apps/dep-nginx created
$ 
$ 
$ 
$ 
$ kubectl get pods -o wide --show-labels
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES   LABELS
dep-httpd-596bb69fc4-hsbn6   1/1     Running   0          25s   172.17.0.7    minikube   <none>           <none>            component=httpd_pod,pod-template-hash=596bb69fc4
dep-httpd-596bb69fc4-knmqv   1/1     Running   0          25s   172.17.0.8    minikube   <none>           <none>            component=httpd_pod,pod-template-hash=596bb69fc4
dep-httpd-596bb69fc4-n84s8   1/1     Running   0          25s   172.17.0.9    minikube   <none>           <none>            component=httpd_pod,pod-template-hash=596bb69fc4
dep-nginx-5965bddb9c-k2r66   1/1     Running   0          18s   172.17.0.12   minikube   <none>           <none>            component=nginx_pod,pod-template-hash=5965bddb9c
dep-nginx-5965bddb9c-kg6j7   1/1     Running   0          18s   172.17.0.10   minikube   <none>           <none>            component=nginx_pod,pod-template-hash=5965bddb9c
dep-nginx-5965bddb9c-knb4d   1/1     Running   0          18s   172.17.0.11   minikube   <none>           <none>            component=nginx_pod,pod-template-hash=5965bddb9c
```

Now let's create our 2 ClusterIPs:

```bash
$ kubectl apply -f configs/eg1-ingress/svc-ClusterIP-httpd.yml
service/svc-clusterip-httpd created
$ kubectl apply -f configs/eg1-ingress/svc-ClusterIP-nginx.yml
service/svc-clusterip-nginx created


$ kubectl get svc -o wide
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP    3d21h   <none>
svc-clusterip-httpd   ClusterIP   10.101.204.16   <none>        4000/TCP   20s     component=apache_webserver
svc-clusterip-nginx   ClusterIP   10.101.250.15   <none>        5000/TCP   11s     component=nginx_webserver
```

These deployments and ClusterIP objects are stuff we've covered before, so there's nothing new here. It's the next part that's new, which is that we create the Ingress object: 


```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  rules:
    - host: httpd-demo.com    # we specify a domain name this time. 
      http:
        paths:
          - path: /
            backend:
              serviceName: svc-clusterip-httpd
              servicePort: 4000
    - host: caddy-demo.com          # we specify a domain name this time. 
      http:
        paths:
          - path: /
            backend:
              serviceName: svc-clusterip-caddy
              servicePort: 2015
```

There are a lot of [Ingress Controller Annotation settings](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md) available that allows you to customise your Kubernetes setup. This yaml file creates:


```bash
$ kubectl get ingresses -o wide
NAME              HOSTS                           ADDRESS     PORTS   AGE
ingress-service   httpd-demo.com,caddy-demo.com   10.0.2.15   80      105s


$ kubectl describe ingresses
Name:             ingress-service
Namespace:        default
Address:          10.0.2.15
Default backend:  default-http-backend:80 (172.17.0.5:8080)
Rules:
  Host            Path  Backends
  ----            ----  --------
  httpd-demo.com  
                  /   svc-clusterip-httpd:4000 (<none>)
  caddy-demo.com  
                  /   svc-clusterip-caddy:2015 (<none>)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"kubernetes.io/ingress.class":"nginx","nginx.ingress.kubernetes.io/force-ssl-redirect":"false","nginx.ingress.kubernetes.io/rewrite-target":"/","nginx.ingress.kubernetes.io/ssl-redirect":"false"},"name":"ingress-service","namespace":"default"},"spec":{"rules":[{"host":"httpd-demo.com","http":{"paths":[{"backend":{"serviceName":"svc-clusterip-httpd","servicePort":4000},"path":"/"}]}},{"host":"caddy-demo.com","http":{"paths":[{"backend":{"serviceName":"svc-clusterip-caddy","servicePort":2015},"path":"/"}]}}]}}

  kubernetes.io/ingress.class:                     nginx
  nginx.ingress.kubernetes.io/force-ssl-redirect:  false
  nginx.ingress.kubernetes.io/rewrite-target:      /
  nginx.ingress.kubernetes.io/ssl-redirect:        false
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  2m24s  nginx-ingress-controller  Ingress default/ingress-service
  Normal  UPDATE  106s   nginx-ingress-controller  Ingress default/ingress-service

```

Now we're ready to test this out, from our macbook we run:

```bash
$ minikube ip
192.168.99.105
$ curl http://192.168.99.105
default backend - 404
```

This is the nginx forward proxy reporting an error, (this error message comes from the 'Default backend' specified above, which is also configurable). It errored because in our example, the nginx-forward proxy only works when the request is made using a dns url. These urls are just our demo urls which are not available on any public dns servers, we have to therefore simulate them, which we can do using curl like this:

```bash
$ curl http://192.168.99.105 -H "Host: caddy-demo.com"
Hello I'm the caddy pod, dep-caddy-68df997cb7-4gs5c, and I'm displaying this page.
$ curl http://192.168.99.105 -H "Host: caddy-demo.com"
Hello I'm the caddy pod, dep-caddy-68df997cb7-9mxjl, and I'm displaying this page.


$ curl http://192.168.99.105 -H "Host: httpd-demo.com"
The is the Apache/httpd pod, dep-httpd-64f4d946d7-ccfwn, which is displaying this page.
$ curl http://192.168.99.105 -H "Host: httpd-demo.com"
The is the Apache/httpd pod, dep-httpd-64f4d946d7-8xssp, which is displaying this page.
```

Notice here that the loadbalancing feature is also working. 


You can simulate this urls by adding the demo urls into our local hosts file:

```bash
echo "$(minikube ip)   caddy-demo.com" >> /etc/hosts
echo "$(minikube ip)   httpd-demo.com" >> /etc/hosts
```

Now we can retest with a simpler curl command

```bash
$ curl http://httpd-demo.com
The is the Apache/httpd pod, dep-httpd-64f4d946d7-8xssp, which is displaying this page.
$ curl http://httpd-demo.com
The is the Apache/httpd pod, dep-httpd-64f4d946d7-ccfwn, which is displaying this page.
$ curl http://caddy-demo.com
Hello I'm the caddy pod, dep-caddy-68df997cb7-4gs5c, and I'm displaying this page.
$ curl http://caddy-demo.com
Hello I'm the caddy pod, dep-caddy-68df997cb7-9mxjl, and I'm displaying this page.
```

Success!



### Further Reading

This is to do with:


https://github.com/kubernetes/ingress-nginx

https://kubernetes.github.io/ingress-nginx/


https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html

https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-1-d1ede3322727
https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-2-13fdc6c4e24c
https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-3-f35957784c8e

https://medium.com/@awkwardferny/getting-started-with-kubernetes-ingress-nginx-on-minikube-d75e58f52b6c

https://www.reddit.com/r/kubernetes/comments/8x1am5/get_automatic_https_with_lets_encrypt_and/

https://kubedex.com/ingress/