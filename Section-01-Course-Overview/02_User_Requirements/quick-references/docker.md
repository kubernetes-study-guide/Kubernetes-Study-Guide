```
$ docker system info
$ docker pull hello-world
$ docker image ls
$ docker run hello-world
$ docker container create --name cntr_hello-world hello-world
$ docker container ls --all
$ docker ps --all
$ docker container start --attach cntr_hello-world 
$ docker run -it ubuntu bash
$ docker logs thirsty_hopper
$ docker run httpd pwd
$ docker ps --all
$ docker start -a cntr_hello-apache echo goodby
$ docker container stop cntr_name
$ docker container rm cntr_name
$ docker container stop $(docker container ls --all --quiet) 
$ docker container rm $(docker container ls --all --quiet) 
$ docker image rm $(docker image ls --quiet)
$ docker system prune --all --volumes --force
$ docker container ls --all
$ docker container run centos:latest
$ docker run --detach httpd
$ docker run --detach -p 80:80 httpd
$ docker container run -it centos:latest /bin/bash
$ docker run -it httpd /bin/bash
```