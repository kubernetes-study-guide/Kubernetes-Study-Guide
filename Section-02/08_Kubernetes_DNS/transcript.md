# Kubernete's DNS 


Hello everone and welcome back. 

We're now going to talk about how to use DNS in Kubernetes. But before let me quickly explain what dns is. 


When you access a website, then you access it by using it's website address, rather than it's ip address. For example you don't access Google by using the google server's ip address, instead you use the domain name google.com. The ability to use domain names is made possible thanks to DNS servers. Domain Name Servers (DNS) are the Internet's equivalent of a phone book. Where instead of a person's name and their phone number, we have a domain name and it's IP address. So when you enter a domain name, Your web browser still uses ip addresses behind the scenes, by first querying a dns server to find out what IP number to use for a given domain name. The dns server that your web browser queries is defined somewhere on your machine. For example on Linux and Mac systems, it's defined in your resolv.conf file"

```bash
/etc/resolv.conf.
```

In my case it's showing my dns server as my local router's ip address, and my router in turn queries one my internet service provider's dns servers. You can also perform manual dns lookups using the nslookup command. 

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












The reason we use domain names in ther first place is because IP addresses can change overtime and are harder to remember.


The same is true for pods. Pod to pod communication should be done using dns names, not ip addresses. Luckily Kubernetes comes with a builtin internal DNS service, in the form of coredns. 

In some of the previous videos we have demoed how one pod can reach another pod by curling the ip address. However communicating with IP addresses is bad practice. Instead you should communicate using dns names.
For example we want to access facebook, we don't access if by entrering facebook's ip address, instead we access it using facebook's dns name, which is facebook.com. 

There's 2 obvious reasons why dns is better than ip address, first dns names are easier to remember, and secondly ip addresses can change over time. 



Kubernetes comes with a builtin internal dns service. 


In the previous video we run a curl command from one pod to another pod


kubernetes comes with builtin dns. This builtin dns only stores dns records for service objects. 


So here's the curl command you can use: 

```bash

```

This domain name is made up of a few parts:

- the service's name
- namespace name
- the base domain name. 


So whenever you want one pod to access another pod, then use domain names like this. Don't use ip addresses. 


Now you might be wondering how this dns works behind the scenes. 



nslookup

dig

For example you can run the following commands to query the builtin dns.  So did know where to access the dns service?









That's because the ip address can change, whenever pod is rebuilt. The recommend approach is to access pods using domain names.


It's bad practive to use IP addresses to access things. 


the same approach that we use for rest of the world wide web, and that's to use dns 



This kubernetes service ip address will actually never change. That's because this ip address is hardcoded into the kube clusters configuration files. The only way to change this ip address is by deliberately going into your kube cluster's config files and then changing it there. 





