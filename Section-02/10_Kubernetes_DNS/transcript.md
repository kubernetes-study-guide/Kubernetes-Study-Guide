Hello everyone one, and welcome back.

The recommended approach is to use a Domain Name Service, or DNS for short. Lets take a quick detour and talk about DNS more generally. DNS is quite a big subject to cover in this course, but the basic premise of DNS is that it's a system that let's you assign a name to an IP address. It's a bit like how you assign a name to a phone number in your smart phone's address book. The World Wide Web relies on DNS in order to work, becuase it's used for resolving website addresses to ip addresses. 

You can perform DNS lookups from the command line using nslookup. For example let's do a DNS lookup of my website, codingbee.net:

```bash
$ nslookup codingbee.net
Server:         194.168.4.100
Address:        194.168.4.100#53

Non-authoritative answer:
Name:   codingbee.net
Address: 77.104.171.177
```

Here it says that codingbee.net resides on the server with the ip address 77.104.171.177.

Now let's 



That's why in the real world, we use DNS instead of raw ip addresses. For example when you want to access Google, you use the dns name of google.com. You don't use the google's IP address. 

you don't do that by typing the google server's ip address into your web browser, instead you etner it's dns name, google.com. 

In Kubernetes, you can also create your own easy to remember dns names for your pods. That's done by creating Service objects. Let's demo kubernetes dns by creating a nodeport service. 

```





With service object you can access a pod via the service object, rather than using the pod's ip address. Let's demo this by creating a nodeport service, here's the yaml file I'll use to create nodeport service:

```bash
code ...
```

Once again I've taken this yaml file from an earlier demo, where we've set port xx for pod-to-pod communication. So let's now go ahead and create this service:


```bash
kubectl apply -f ...
```

As you can see, service objects comes with it's own ip address. So let's try curling the services ip address:

```bash
kubectl exec -- curl....
```

Here we can see that our service object forwarded our curl request to the apache pod, which then sent the response. That's because this service and apache pod are associated with each other through label and selectors, as covered in earlier videos. One way to confirm this association is by using the endpoints command:

```bash
$ kubectl get endpoints svc-nodeport-httpd
NAME                 ENDPOINTS       AGE
svc-nodeport-httpd   172.17.0.3:80   8s
```

Here we can see our service will forward traffic to the following ip address at this very moment in time. This info get's update in realtime. 

Now we no longer have to worry about the a pod's ip number changing, because if it does change the our service will automatically update update the endpoint with the new pod ip address. 




There's a lot more to services than just setting up dns and we'll explore them later. 



and a centos pod. 

now let's curl by id address. Now lets do the same thing but using the nodeport service objects, 

then lets curl the service ip address. 

So far so good. 

However there a few problem with use ip address:

- ip address can change
- ip address are not informative. a dns name google.com is informative. 
- dns names have a logical naming structure that you can specify in your pods at creation time.  

To fix this, we can use kubernetes dns. That's something that comes preinstalled by default in most installer options, such as minikube. 

create another new, shorter article after this, called : clusterip services. 
