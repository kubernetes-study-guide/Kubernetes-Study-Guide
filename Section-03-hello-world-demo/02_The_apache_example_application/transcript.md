We'll use a simple web server as our example application for the next few sections of this couurse. 

I'll be runnign this applcation inside a container and I'll be using the ubiquitous apache webserver as the software to run my example web server app.

I'll be creating my container using the official apache docker hub image, called httpd. However I tend to refer to this image as just the apache image. 


Let's take a look at our example containeer in action:

```
$ docker run -d -p 8080:80 httpd
```

Here I'm instructing to spin up a new container using httpd image. set up portforwarding so that any traffic i send from my workstation at port 8080 get's forward to the container's port 80, since the apache image is preconfigured to listen on port 80. I want this container to run in the background which is why I've enabled the detached mode. 


```
$ docker container ls
```


```
$ docker logs kind_chaplygin 
```

Now let's see what this looks like:

```
$ curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

