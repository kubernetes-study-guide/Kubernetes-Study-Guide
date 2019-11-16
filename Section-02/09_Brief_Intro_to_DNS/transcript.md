Hello everyone one, and welcome back.

In this video I'm going to take a small detour and give a quick intro to DNS. If you're already familiar with what DNS is and how it works, then you can skip this video. 


The basic premise of DNS is that it's a system that let's you assign a website address (aka domain name) to an IP address. 

It's a bit like how you assign a name to a phone number in your smart phone's address book. DNS is used as the internet's address book, where each DNS entry is a website address along with a ip address. There are thousands of DNS servers dotted around the world that provides DNS lookup service. 

You can perform DNS lookups from the command line using dns querying cli tools such as nslookup. For example let's do a DNS lookup of my website, codingbee.net:

```bash
$ nslookup codingbee.net
Server:         194.168.4.100
Address:        194.168.4.100#53

Non-authoritative answer:
Name:   codingbee.net
Address: 77.104.171.177
```

Here it says that codingbee.net resides on the server with the ip address of 77.104.171.177. The DNS server that provided this information, has the ip address of 194.168.4.100. Here we can see our DNS server is listening on port 53 for DNS lookup requests. That's actually the standard port for DNS. Another command you can use for performing dns lookups is dig:

```bash
$ dig codingbee.net

; <<>> DiG 9.10.6 <<>> codingbee.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28099
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;codingbee.net.                 IN      A

;; ANSWER SECTION:
codingbee.net.          10538   IN      A       77.104.171.177

;; Query time: 18 msec
;; SERVER: 194.168.4.100#53(194.168.4.100)
;; WHEN: Wed Nov 13 22:25:58 GMT 2019
;; MSG SIZE  rcvd: 58
```


So you might be wondering how did dig and nslookup know about the DNS server's IP address is in the first place? Well in the case of Linux and Apple Macs, the DNS server's ip address is stored in the etc-resolv.conf file:



```bash
$ cat /etc/resolv.conf
```

Here we have 2 dns server just in case one of them fail. These DNS servers are actually provided by my Internet service Provider. 

These IP addresses are automatically updated everytime you connect to an internet network, such as your home wifi network. For example if I disconnect from my wifi, then the /etc/resolv.conf file gets deleted as well. 

```bash
perform gui action. 
```

Now there can be times when you want to create your own personal dns entry for testing purposes. That's actually possible by adding your entry to the etc-hosts files:

```bash
cat /etc/hosts
```

For example my website's domain name, codingbee.net currently resolves to the actual server that codingbee.net is running on. So if I run a curl request then I get a response from the actual server: 

```bash
$ curl http://codingbee.net 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://codingbee.net/">here</a>.</p>
</body></html>
```

Don't worry too much about what it says here, it's just the webserver saying to use https instead of http.

Now let's say I want to resolve codingbee.net to my local workstation. Then to do that I add the following to the etc/hosts file:

```
vim /etc/hosts
```

Tools such as google chrome, firefox, curl and wget, will always first look in the hosts file for a match before they query dns. So if they find a match in the hosts file, then it won't bother with doing a dns lookup. That means that adding an entry in your hosts file will effectively override dns. Ok I've added my custom entry, Now if I repeat the curl request I now get:

```
$ curl http://codingbee.net 
curl: (7) Failed to connect to codingbee.net port 80: Connection refused
```

This time it has failed, becuase I don't have anything on my workstation that's listening on port 80. SO I'll quickly spin up a dummy local web server using docker and configure it to listen on port 80:


```bash
$ docker run -p 80:80 httpd

```

Now while that's running I'll open up a new bash terminal and retry the curl test:

```bash
$ curl http://codingbee.net 
```

This time I got a successful response. 


So far I've only scratched the surface of DNS. But hopefully I've covered enough to help you continue with the rest of this course. 

That's it for this video, see you in the next one. 


=====================================================


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