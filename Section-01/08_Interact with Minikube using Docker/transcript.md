Hello everyone. 

In minikube, docker is used as the kubeclusters default container run time engine.

In case you're interested, you can  configure your workstation's docker cli to connect to the minikube's docker deamon. To do that, you first run the docker-env command:

```bash
minikube docker-env
```

This command just echos out info about what you need to do to configure your docker cli. This output also give you a one-liner setup command which you can use instead:

```bash
eval $(minikube docker-env)
```

This change only lasts for the current bash session. Now if you do a docker ls, you should see something like this:

```bash
$ docker container ls
```

Notice that we already have a lot of containers running. These containers are actually used internally by kubernetes itself. 

If you don't have docker installed on your workstation, then you can use the minikube VM's docker cli instead. 

To do that, you first have to ssh into the minikube VM, and then try using docker:

```bash
$ minikube ssh

$ docker container ls
```

That's it for this video. See you in the next one. 
