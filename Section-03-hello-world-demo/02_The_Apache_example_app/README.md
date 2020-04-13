# Course Notes - The Apache example Application


# Apache Web Server
Here are some places where you can learn more about Apache

[Official Apache Website](https://httpd.apache.org/)
[Official Apache DockerHub image](https://hub.docker.com/_/httpd)
[Guide on installing & Configuring Apache Web Server](https://www.digitalocean.com/community/tutorial_collections/21) on various Linux Distros.



## Commands used in this video

```
$ docker run -d -p 80:80 httpd
```

In the above command I'm instructing docker to spin up a new container using the official apache image. I've setup portforwarding so that any traffic i send from my workstation using localhost and port 80, get's forward to the container's port 80, since that's the default port that's baked into the apache image. I want this container to run in the background which is why I've enabled the detached mode. Let's now check our container's status


This image's name, httpd, is actually the same name as the name of the process that runs when you start an apache web server, which is also httpd. It's short for http daemon.