We'll use a simple web server as our example application for the next few sections of this couurse. 

I'll be runnign this applcation inside a container and I'll be using the ubiquitous apache webserver as the software to run my example web server app.

I'll be creating my container using the official apache docker hub image, called httpd. However I tend to refer to this image as just the apache image. 


Let's take a look at our example containeer in action:

```
$ docker run -d -p 80:80 httpd
```

I'll quickly go over this in your case your docker knowledge is a bit rusty. Here I'm instructing docker to spin up a new container using the official httpd image. I've setup portforwarding so that any traffic i send to localhost using port 80 get's forward to the container's port 80, since that's the default port that's backed into the apache image. I want this container to run in the background which is why I've enabled the detached mode. Let's now check our container's status


```
$ docker container ls
```

Ok it looks like our containers up and rurning. I'll also check the container's logs:

```
$ docker logs kind_chaplygin 
```

So far so good, everything looks ok here. Also notice that the container's primary process was started by executing the httpd command. That's actually where this image's name originates from. 

Now let's test this container using curl:

```
$ curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

Our container is basically serving up the content's of the container's homepage, which has some default static html content which we can see above.

You can also see the same thing if you open it up in a browser. It's still showing the same content but this time web browser has rendered the html content. 

Ok so that's the containerised application I'll be using as my example for the next few sections. 