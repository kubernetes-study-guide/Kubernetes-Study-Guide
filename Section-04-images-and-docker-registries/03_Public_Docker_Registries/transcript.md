In most workplaces docker is primarily used for building images using the docker build command along with a docker file. And result docker images are pushed up to docker registries. 

Once the images are in a docker registries, the images can then be downloaded by a kubernetes cluster.

A lot of docker registries lets you password protect your images, in other words make your images private. If you want to restrict access to only authenticated requesters. Kubernetes is able to download both public and private images. Although to get Kubernetes to pull down private images, you need to provide Kubernetes with valid login credential. We'll show you how to do that later in the course. 

There are quite a few public docker registries you can make use of. The defacto is dockerhub, but there are other popular alternatives:

- docker.io (aka dockerhub)
- github.com
- https://catalog.redhat.com/software/containers/search


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

xxxxxxxxxxxxxxxxxxxxxxx

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


