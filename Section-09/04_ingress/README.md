# Ingress

Earlier we covered service objects which sets up networking between pods in the same kubecluster, by creating a ClusterIP service, and also how we can make a pod externally accessible by creating a NodePort service. However as mentioned, Nodeport shouldn't be used in a production environment.

The ideal solution would be to have your worker nodes only listening to standard ports, e.g. port 443 for https. That's possible by setting up Ingress objects.

Ingress ojects are used to make pods externally accessible. Before you can start creating ingress objects, you first need to setup an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). Ingress controllers provides Forward Proxy, 'Edge router' and 'Loadbalancing' features. One popular choice is [Traefik](https://github.com/containous/traefik). However in our case we'll use [ingress-nginx github repo](https://github.com/kubernetes/ingress-nginx), which essentially makes use of an internal nginx forward-proxy pod behind the scenes.

The way you set up the ingres-nginx controller depends on what underlying platfrom you're using, which in our case is minikube.

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

This creates a new namespace and populates it with some objects.

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
  annotations:   # this is something specific to ingress objects. It lets you customise your ingress setup.
    kubernetes.io/ingress.class: nginx   # this is how we tell k8s to build ingress controller using the nginx project.
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  rules:
    - host: httpd-demo.com
      http:                    # this enables listening on port 80, i.e. http port
        paths:
          - path: /
            backend:
              serviceName: svc-clusterip-httpd
              servicePort: 4000
    - host: caddy-demo.com
      http:
        paths:
          - path: /
            backend:
              serviceName: svc-clusterip-caddy
              servicePort: 2015
```

There are a lot of [Ingress Controller Annotation settings](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations) available that allows you to customise your Kubernetes setup.

Also here are some useful links:

- [https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/basic-usage.md](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/basic-usage.md)
- [https://github.com/kubernetes/ingress-nginx/blob/master/deploy/mandatory.yaml](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/mandatory.yaml)
- [https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

Applying this yaml results in:

```bash
$ kubectl get ingress
NAME              HOSTS                           ADDRESS     PORTS   AGE
ingress-service   httpd-demo.com,caddy-demo.com   10.0.2.15   80      2m54s
```

Here's more detailed info:

```bash
$ kubectl describe ingress ingress-service
Name:             ingress-service
Namespace:        default
Address:          10.0.2.15
Default backend:  default-http-backend:80 (172.17.0.3:8080)
Rules:
  Host            Path  Backends
  ----            ----  --------
  httpd-demo.com
                  /   svc-clusterip-httpd:4000 (<none>)
  caddy-demo.com
                  /   svc-clusterip-caddy:2015 (<none>)
Annotations:
  kubernetes.io/ingress.class:                       nginx
  nginx.ingress.kubernetes.io/force-ssl-redirect:    false
  nginx.ingress.kubernetes.io/rewrite-target:        /
  nginx.ingress.kubernetes.io/ssl-redirect:          false
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{"kubernetes.io/ingress.class":"nginx","nginx.ingress.kubernetes.io/force-ssl-redirect":"false","nginx.ingress.kubernetes.io/rewrite-target":"/","nginx.ingress.kubernetes.io/ssl-redirect":"false"},"name":"ingress-service","namespace":"default"},"spec":{"rules":[{"host":"httpd-demo.com","http":{"paths":[{"backend":{"serviceName":"svc-clusterip-httpd","servicePort":4000},"path":"/"}]}},{"host":"caddy-demo.com","http":{"paths":[{"backend":{"serviceName":"svc-clusterip-caddy","servicePort":2015},"path":"/"}]}}]}}

Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  60m   nginx-ingress-controller  Ingress default/ingress-service
  Normal  CREATE  60m   nginx-ingress-controller  Ingress default/ingress-service
  Normal  UPDATE  60m   nginx-ingress-controller  Ingress default/ingress-service
  Normal  UPDATE  60m   nginx-ingress-controller  Ingress default/ingress-service
```

This shows that we have to now use a dns names to access our pods externally. So we can mimic this by adding entries to our macbooks hosts file:

```bash
sudo echo "$(minikube ip)   caddy-demo.com" >> /etc/hosts
sudo echo "$(minikube ip)   httpd-demo.com" >> /etc/hosts
```

Now we can retest with a simpler curl command

```bash
$ curl http://caddy-demo.com
You've hit caddy pod, dep-caddy-78899cd4cb-5dcq6
$ curl http://caddy-demo.com
You've hit caddy pod, dep-caddy-78899cd4cb-dldc8
$ curl http://httpd-demo.com
You've hit httpd pod, dep-httpd-699cccd58d-2tp5s
$ curl http://httpd-demo.com
You've hit httpd pod, dep-httpd-699cccd58d-bm499
```

In case you don't want to update your /etc/hosts file, then here's another way to test this, by specifying the dns name in the request's header:

```bash
$ curl http://$(minikube ip) -H "Host: caddy-demo.com"
You've hit caddy pod, dep-caddy-78899cd4cb-5dcq6
$ curl http://$(minikube ip) -H "Host: caddy-demo.com"
You've hit caddy pod, dep-caddy-78899cd4cb-dldc8
$ curl http://$(minikube ip) -H "Host: httpd-demo.com"
You've hit httpd pod, dep-httpd-699cccd58d-2tp5s
$ curl http://$(minikube ip) -H "Host: httpd-demo.com"
You've hit httpd pod, dep-httpd-699cccd58d-bm499
```

The important thing here is that we:

- no longer need to specify a port number in the url, since we're now using standard port numbers.

- We can now externally access the pods, and have them loadbalanced as well.

## Missing section: ingress tls

Need to talk about how you would setup ingress to accept https connections. Also see `ingress.spec.tls`.

## Types of routing

ingress provides to types of routing:

- name based hostnames - e.g. send example.com traffic to clusterIP service. 
- paths based routing.  e.g. send example.com/v1 and example.com/v2 traffic to different clusterIP services. 

You can also use a combination of these two approach. 

In order to do this you need to, you need to set up a loadbalancer that forwards traffic to all the cluster's worker nodes. You also need to create multiple dns cname entries for the hostnames to point to the elb's hostname. 


### Further Reading

This is to do with:

[https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)

[https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

[https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html](https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html)

[https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-1-d1ede3322727](https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-1-d1ede3322727)
[https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-2-13fdc6c4e24c](https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-2-13fdc6c4e24c)
[https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-3-f35957784c8e](https://itnext.io/an-illustrated-guide-to-kubernetes-networking-part-3-f35957784c8e)

[https://medium.com/@awkwardferny/getting-started-with-kubernetes-ingress-nginx-on-minikube-d75e58f52b6c](https://medium.com/@awkwardferny/getting-started-with-kubernetes-ingress-nginx-on-minikube-d75e58f52b6c)

[https://www.reddit.com/r/kubernetes/comments/8x1am5/get_automatic_https_with_lets_encrypt_and/](https://www.reddit.com/r/kubernetes/comments/8x1am5/get_automatic_https_with_lets_encrypt_and/)

[https://kubedex.com/ingress/](https://kubedex.com/ingress/)
