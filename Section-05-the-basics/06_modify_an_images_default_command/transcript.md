Hello everyone and welcome back.

in the last video we introduced you to a pod's 'command' and 'args' settings, and we demoed these settings using a centos image.

But I wanted to show you another example but this time using the apache web server image, httpd.

In this demo, I want to change the default message you get when you curl a httpd based pod.

Earlier we saw that the default message is ""

I spent a bit of time to go through this image's official git repo and documentation as well as reading up about the httpd software in general. From my research I found out that this default message is actually the content of the index.html file that the httpd image comes with. We can confirm that by looking inside the image:

```
kubectl run -it ...
cat ...
```

As you can see this is identical to the default message we get when we curled the webserver pod in our hello world demo. That means to change the default message, we need to change the content of this file. There's a few ways to do that, but for this demo I'm going to do it using the commands and args setting. So here's the yaml file I'll use to give this a try:

```
cat ...
```

This is pretty much the same as the yaml file i used in the hello world pod demo. The only difference here is that I've added the command and args section to overwrite the contents of the index.html file.

Ok we're ready to try this file, but first let's bring a up list of pods
```
watch kubectl get pods -o wide
```

At the moment we don't have any pods, now let's try this yaml file.

```
kaf ...
```

Ok...that didn't go to plan. It looks like the pod is failing to start up properly and Kubernetes is keeps trying to restart. Ah, I think I know why that is. It's because by adding in the command and args settings, I've effectively replaced whatever the apache image's default startup command is, with this shortlived command. So what is the apache image's default startup process, one way to find that out is by using docker inspect:

```
docker inspect httpd:latest  # then scroll up
```

Ok it looks like it's this httpd-foreground command. This must be the command that starts up the actual apache web service.  So I still want this command to run, but only after the index.html is updated. So let's append this command to the end of our startup script.

```
use vim to update the file
```

Ok let's try it again:

```
kaf -f ....
```

Cool, the pod is now running.

Let's now test it out, by using a client testpod:

```
tp ....
```

Awesome that worked. Also notice that the hostname variable has resolved to the pod's name.

There are other ways of updating the index.html file, and we'll cover them later in the course. But for now I wanted to show you this approach just to give you an idea on how you might want to use this commands and args settings to tweak an image's original default command.

That's it for this video.

