We're now nearly ready to do our first kubernetes hello world demo, but before that I need to give an overview of the example app we'll be using for the demo.

We'll use a simple web server for our demo application. I'll be using the ubiquitous apache http web server software to run my web server. I'm using a apache because about 40% of all the worlds webservers runs on apache so there's a good chance you're already familiar with this software. If not then head over to the course notes where you can find more info about it.


In our demo, we'll be running apached inside a container. To do that I'll use the official apache docker hub image, called httpd. By the way, I tend to call this image the apache image only because calling it "httpd" all the time is a bit of mouthful.

Now let's a quick look at this apache container in action using docker:

```
$ docker run -d -p 80:80 httpd
```

I'll quickly go over this command in case your docker skills are a bit rusty. Here I'm instructing docker to spin up a new container using the official apache image. I've setup portforwarding so that any traffic i send from my workstation using localhost and port 80, get's forwarded to the container's port 80, since that's the default port apache has been configured to listen on for this image. I want this container to run in the background which is why I've enabled the detached mode. Let's now check our container's status


```
$ docker container ls
```

Ok it looks like our container is now up and running. I'll also check the apache's logs and start following it as well:

```
$ docker logs -f kind_chaplygin   # right terminal
```

Ok, everything looks good here. We can see here that The container's primary process was started by executing the httpd command.

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

By the way I've found the path of this index.html file after reviewing the apache images documentation and configurations.

Now let's say I want to change this message to something else. In that case, we can simly update the index.html file with the new content:

``` right terminal
docker exec -it vigilant_satoshi bash
echo 'hello Sher!!!' > /usr/local/apache2/htdocs/index.html
```

Now if we run curl again we get the new response:

```
curl http://localhost
```

Ok so that's the containerised application I'll be using for my example app. As we progress through this course I'll be showing you how to run this container on kubernetes and then gradually build on that example by showing how you can customise and configure it, make it highly available, and so on.


