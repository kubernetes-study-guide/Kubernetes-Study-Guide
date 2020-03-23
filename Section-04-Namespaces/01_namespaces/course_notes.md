A minikube cluster happens to use Docker as it's default container run time engine.

```bash
$ minikube start --help |  grep container-runtime
      --container-runtime string          The container runtime to be used (docker, crio, containerd) (default "docker")
```

So if you stick with this default then you can run docker commands against your    cluster, which is pretty cool becuase it let's you see what's going on behind the scenes. Before you can run Docker commands, you first need to configure your workstation's docker cli to point to the minikube vm's docker daemon. To help you with that, you can use the docker-env command:

```bash
$ clear
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/schowdhury/.minikube/certs"
export DOCKER_API_VERSION="1.35"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

docker-env doesn't make any changes itself, instead it just tells you what configurations you need to set. This happens to involve running a few export commands in order to create some environment variables . By the way if you don't have docker installed on your workstation, then you can still use docker, by ssh'ing into the minikube vm, where you'll find docker is already installed and configured for you. 


In my case I want to use my workstation's docker cli, and the last line of this output shows how to run all these export commands with a single command:


```bash
eval $(minikube docker-env)
```

Ok we're now connected to minikube's docker daemon for the rest of this bash session. So let's try listing our containers:

```bash
$ clear
command+K
$ docker container ls
scroll up
```

Here we can see that we have a lot of containers, so that means that we must have   some pods running as well. 

```bash
$ kubectl get pods
No resources found.
```

Ok that's strange, we don't have any pods at all. How is that possible, where have all these containers come from?

The answer is to do with namespaces