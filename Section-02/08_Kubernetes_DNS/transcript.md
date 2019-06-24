In the previous video we run a curl command from one pod to another pod


kubernetes comes with built in dns.


It's bad practice to access pods using it's ip address, that's because ip addresses can change. Instead you should do the same thing that you do when surfing the internet, which is to use domain names. For example we don't access google by entering the google server's ip address into our web browser, instead we use the domain name, google.com. And you should take the same approach when accessing pods. 

Kubernetes comes with builtin dns. This builtin dns only stores dns records for service objects. So here's the curl command you can use: 

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





