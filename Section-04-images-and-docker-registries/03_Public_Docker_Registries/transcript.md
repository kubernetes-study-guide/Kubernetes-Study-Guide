Hello everyone one, and welcome back.

So far we've seen that kubernetes has use docker images in order to build it's pods, and kubernetes pulls down these images from docker registries.

There's actually a number of docker registries you can use:

- github
- redhat
- ...

To pull down images from any of these places, you need to specify the following naming convention:

Use, github as an example, might be easier.


<registry-address>/<account-name>/<image-name>:<tag-version>


Here the account-name can be the name of an organisation, or person. The image name also doubles up as the docker registry's repo name. So this particular repo, called ubi, houses multiple version of the image called ubi. Finally we have the tag name, where you can set which version of the image you want to pull down from this image repo.



Here's an example:

```
$ kubectl run --rm=true -it client --image=registry.redhat.io/ubi8/ubi:8.0 --restart=Never -- cat /etc/redhat-release
```

That's odd, it looks like somethings gone wrong. ok let me describe the pod


Here's an example,

```
$ kubectl run --rm=true -it client --image=registry.access.redhat.com/ubi8/ubi:8.0 --restart=Never -- cat /etc/redhat-release
```

Here I'm pulling down the ubi image. I'll talk about this image in the next video.

Now what if I want install the latest version of particular image. Then by convention the latest version has the tag-version set to 'latest'.

```
$ kubectl run --rm=true -it client --image=registry.access.redhat.com/ubi8/ubi:latest --restart=Never -- cat /etc/redhat-release
```

However, the tag version setting is optional, so if you don't explicitly specify it then it will default to latest anyway:

```
$ kubectl run --rm=true -it client --image=registry.access.redhat.com/ubi8/ubi --restart=Never -- cat /etc/redhat-release
```

In this instance that at this present time, the tag latest is just an alias for 8.1, since 8.1 is the latest available version.


Dockerhub is a public docker registry, but in it you can have private image repos. Meaning that you would need provide Kubernetes with login credentials to authenticate against dockerhub before it can do an image pull. I'll cover how do that later in the course.











A lot of docker registries lets you password protect your images, in other words make your images private. If you want to restrict access to only authenticated requesters. Kubernetes is able to download both public and private images. Although to get Kubernetes to pull down private images, you need to provide Kubernetes with valid login credential. We'll show you how to do that later in the course.

There are quite a few public docker registries you can make use of. The defacto is dockerhub, but there are other popular alternatives:

- docker.io (aka dockerhub)
- github.com
- https://catalog.redhat.com/software/containers/search

more info:
list of ubi images available:
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index?extIdCarryOver=true&sc_cid=701f2000001OH7JAAW#get_ubi_images

https://developers.redhat.com/products/rhel/ubi/


Out of all these registries, dockerhub is the king.



Images have the following naming conventions:


There are other public registries you can download images from:
https://catalog.redhat.com/software/containers/search
github.com


UBI is short for Universal Base Image, that's similar os based images, such as the centos image. . It is a general purpose image that you can use as a base image for building your docker image. This image is growing in popularity because:

- developled by redhat
- enterprise grade image.
- totally free.


So how do we use this image?

https://catalog.redhat.com/software/containers/search

registry-name/account-name/my-image-name:tag-name.

Here's an axample for ubi path:

```
$ kubectl run --rm=true -it client --image=registry.access.redhat.com/ubi8/ubi:latest --restart=Never -- bash
```


if the image you want to use has the tag called 'latest'
gd,te



if downlaoding from dockerhub. Then you can get away with omitting the registry-name.

For example I registered a new docker account, and I've chosen codingbee as my username. This username also became my account name.





In our demo we'll be using the following image:
https://hub.docker.com/_/httpd

If you are using on of the official docker hub image


ubuntu
centos
nginx
debian
alpine
mysql
busybox

All these images are official images, with special images you don't . For example I've created a dockerhub account  called 'codingbee' and I've uploaded an image called cb1. So to build an pod with this image. I have to use the account_name-slash-imagename




Images made by the wider community.


There's another image that we'll be using that's not available on dockerhub. There's

ubi - is not found on dockerhub.


Check out the quiz to see examples of how to start up lots of other images.



One image that we'll be using in this course is the official apache web server image, which is call httpd. I'll be using httpd quite a lot in this course so I'm spend a couple of minutes taking a look inside.

Let's start with looking at how this image was made by viewing it's dockerfile. We can locate the dockerfile by first going to the image's official docker hub homepage.

```

```


Awesome that worked. But not only that, unlike in previous demos where we used to get a generic 'it's working' message. This time it printed out the pod's name. That's thanks to the tweak I'm made to the docker image's startup script. The httpd image is preconfigured to just run the httpd-foreground command. I found that out by looking up the httpd image's official Dockerfile on Github. I've modified this slightly by inserting a command to update the index.html file's content, so that it includes the pod's hostname. Pretty cool right.


