# Kubernete's DNS 


Hello everone and welcome back. 

We're now going look at how to use DNS in Kubernetes. But before let me quickly explain what dns is. 


When you access a website, then you access it by using it's website address, rather than it's ip address. For example you don't access Google by using the google server's ip address, instead you use the domain name google.com. The ability to use domain names is made possible thanks to DNS servers. Domain Name Servers (DNS) are the Internet's equivalent of a phone book. Where instead of a person's name and their phone number, we have a domain name and it's IP address. So when you enter a domain name, Your web browser still uses ip addresses behind the scenes, by first querying a dns server to find out what IP number to use for a given domain name. The dns server that your web browser queries is defined somewhere on your machine. For example on Linux and Mac systems, it's defined in your resolv.conf file"

```bash
/etc/resolv.conf
```

In my case, this is my local router's ip address, which means that my local router is also my dns server, however behind the scenes, my router is actually relaying dns requests to one my internet service provider's dns servers. You can also perform manual dns using the nslookup command. 

```bash
$ nslookup codingbee.net
Server:         192.168.0.1
Address:        192.168.0.1#53

Non-authoritative answer:
Name:   codingbee.net
Address: 77.104.171.177
```



Here we can see which nameserver was used and what our website's ip address is. You can also use the dig command to perform dns lookups as well:

```bash
$ dig codingbee.net

; <<>> DiG 9.10.6 <<>> codingbee.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4183
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;codingbee.net.                 IN      A

;; ANSWER SECTION:
codingbee.net.          11070   IN      A       77.104.171.177

;; Query time: 5 msec
;; SERVER: 192.168.0.1#53(192.168.0.1)
;; WHEN: Sat Jun 29 19:20:30 BST 2019
;; MSG SIZE  rcvd: 47
```


It's actually possible to set your own custom dns entries locally on your workstation or container. That's done by adding them to your hosts file:

```bash
cat /etc/hosts
```

We'll demo how to make use of this file in later videos. Now let's turn our attention back to Kubernetes. 


In earlier videos, we demoed pod-to-pod communication by running curl from a cent-os pod to an apache pod by using the apache pod's ip address. However ip addresses are prone to changing, so it's better to use domain names instead. Luckily most Kubernetes installer tools, such as minikube, sets up DNS for you, by installing a DNS software called CoreDNS. You can then set dns entries in coredns by creating service objects.


This domain name is made up of a few parts:

- the service's name
- namespace name
- the base domain name. 





This kubernetes service ip address will actually never change. That's because this ip address is hardcoded into the kube clusters configuration files. The only way to change this ip address is by deliberately going into your kube cluster's config files and then changing it there. 


By the way, you can control the content of your pod's resolv.conf



## Reference
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/ (talks about how to customise pods resolv.conf file - should be covered in a seperate video)





