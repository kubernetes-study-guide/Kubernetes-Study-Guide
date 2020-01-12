Images have the following naming conventions:


There are other public registries you can download images from:
https://catalog.redhat.com/software/containers/search
github.com


registry-name/account-name/my-image-name:tag-name. 

Here's an axample for ubi path:

xxxxxxxxxxxxxxxxxxxxxxx

if the image you want to use has the tag called 'latest'




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


