A common thing a lot of people do when playing with a docker image, is to take a look inside the image. With docker, you can do that by using the docker run command. For example if I want to create a shell session inside the official centos image, then I can run something like this:

```
$ docker run -it --rm --name client centos bash
```

Here I'm telling docker to spin up a container from the centos image and instruct that container to run bash. I've asked docker to name the container "client". I've also requested that my current terminal should be attached to this bash session as an interactive terminal. The rm flag tells docker to delete this container after the bash process ends, which will happen when I exit out of this container. 

Ok before I run this command let's first get a list of all our containers. I'll do that in a another terminal:

```
# open another terminal
watch -n1 docker container ls --all
```

Notice I'm using the 'watch' command here. This utility is used for running a command, over and over again. So in this example I've instructed "watch" to run "docker container ls" every second and show it's output. The watch command's -n flag is where I've set the refresh internal to one second.  This is a simple technique I like to use to monitor what's going on in near realtime. Ok I'll hit enter to get that started. 

```
<enter>
```

At the moment we don't have any containers. Ok let's now run the docker run command:

```
$ docker run -it --rm --name client centos bash <enter>
```

Ok it looks like we now have a running container. 

```
cat /etc/centos-release
```

And since I've 







Ok what if you have an existing container that you want to look inside. Ok let's first show how that's done in docker world before showing the kubernetes way. 







In the quiz section, there are several exercises you can try to create,

using the following types of apps:


mysql
jenkins
wordpress
