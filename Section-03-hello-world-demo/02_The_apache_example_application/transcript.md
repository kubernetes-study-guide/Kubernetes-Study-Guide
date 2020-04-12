We're now nearly ready to do our first kubernetes hello world demo, but before that I need to go over the example app we'll be using for the demo.

I'm going to use a simple web server for our demo application and I'll be using the ubiquitous apache software to run this web server. I'm using apache because about 40% of the world's webservers runs on it, so there's a good chance you're already familiar with it. If not then head over to the course notes where you can find more info about it.


In our demo, we'll be running apache inside a container. To do that I'll use the official apache docker hub image, called httpd. I tend to call this the apache image only because calling it "httpd" all the time is a bit of mouthful.

Now let's take a look at this image in action by spinning up a container with it using docker:

```
$ docker run -d -p 80:80 httpd
```

I'll quickly go over this command just so that we're on the same page. I'm instructing docker to spin up a new container using the official apache image from dockerhub. I've setup portforwarding so that any traffic i send from my workstation using localhost and port 80, get's forwarded to the container's port 80, since that's the port this image has been configure to listen on by default. I want this container to run in the background which is why I've enabled the detached mode. Let's now check our container's status


```
$ docker container ls
```

Ok it looks like our container is now up and running. I'm also going to start following the apache's logs on a different terminal just to keep an eye on things:

```
$ docker logs -f kind_chaplygin   # right terminal
```

Ok, everything looks good here. We can see here that The container's primary process was started by running the httpd command.

Now let's try out this container using curl:

```
curl http://localhost
<html><body><h1>It works!</h1></body></html>
```

ok that worked, Our container is basically serving up the content from the index.html file:

```
docker exec xxxx cat /usr/local/apache2/htdocs/index.html
<html><body><h1>It works!</h1></body></html>
```

I had to go through this image's documentation to figure out where to find this index.html file.

Now let's say I want to change this default message to something else. In that case, we can simply update the index.html file with the new content:

``` right terminal
docker exec -it vigilant_satoshi bash
echo 'hello Sher!!!' > /usr/local/apache2/htdocs/index.html
```

Now if we run curl again we get the new response:

```
curl http://localhost
```

Ok so that's the containerised application I'll be using for my example app. As we progress through this course I'll be showing you how to run this container on kubernetes and then gradually build on that example by showing how you can customise and configure it, make it highly available, and so on.


