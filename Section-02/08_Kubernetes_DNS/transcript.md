In the previous video we run a curl command from one pod to another pod


kubernetes comes with built in dns.


It's bad practice to access pods using it's ip address. That's because the ip address can change, whenever pod is rebuilt. The recommend approach is to access pods using domain names.


It's bad practive to use IP addresses to access things. For example we don't access google by entering the google server's ip address into our web browser, instead we use the domain name, google.com. 


the same approach that we use for rest of the world wide web, and that's to use dns 



This kubernetes service ip address will actually never change. That's because this ip address is hardcoded into the kube clusters configuration files. The only way to change this ip address is by deliberately going into your kube cluster's config files and then changing it there. 





