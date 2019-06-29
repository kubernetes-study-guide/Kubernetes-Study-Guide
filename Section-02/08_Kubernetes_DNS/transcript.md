# Kubernete's DNS 

In some of the previous videos we have demoed how one pod can reach another pod by curling the ip address. However communicating with IP addresses is bad practice. Instead you should communicate using dns.

Domain Name Servers (DNS) are the Internet's equivalent of a phone book. Where instead of a person's name and their phone number, we have a domain name and it's IP address. We normally use dns when accessing a website. For example we want to access facebook, we don't access if by entrering facebook's ip address, instead we access it using facebook's dns name, which is facebook.com. 

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





