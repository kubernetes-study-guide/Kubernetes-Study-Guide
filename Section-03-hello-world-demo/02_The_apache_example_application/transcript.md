We'll use a simple web server as our example application for this course. 


I'll be using the ubiquitous apache http software as my web server software, and I'll be running it inside a container. I'm using a apache because about 40% of all the worlds webservers runs on apache so there's a good chance you've already worked with Apache servers before. If not then head over to the course notes where you can read up about apache webservers.  

To set that up, I'll use the official apache docker hub image, called httpd. By the way, I'm just going to refer to this image as the apache image from now on, only because it's easier then calling it "httpd" all the time which is a bit of mouthful. 


Let's take a look at what our example container looks like. First I'll use docker to spin up container from the apache image. 

```
$ docker run -d -p 80:80 httpd
```

I'll quickly go over this command in case your docker skills are a bit rusty. Here I'm instructing docker to spin up a new container using the official apache image. I've setup portforwarding so that any traffic i send from my workstation using localhost and port 80, get's forward to the container's port 80, since that's the default port that's baked into the apache image. I want this container to run in the background which is why I've enabled the detached mode. Let's now check our container's status


```
$ docker container ls
```

Ok it looks like our container is now up and running. I'll also check the container's logs:

```
$ docker logs kind_chaplygin 
```

So far so good, everything looks ok here. Also notice that the container's primary process was started by executing the httpd command. That's actually where this image gets it's name from. 

Now let's try out this container using curl:

```
$ curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

Our container is basically serving up the content's of the container's homepage, which has some default static html content which we can see above.

You can also see the same thing if you open it up in a browser. It's still showing the same content but this time the web browser has rendered the html content. 

Ok so that's the example containerised application I'll be using. As we progress through this course I'll be showing you how to run this container on kubernetes and then gradually build on that example by showing how you can customise and configure it, make it highly available, and so on. 