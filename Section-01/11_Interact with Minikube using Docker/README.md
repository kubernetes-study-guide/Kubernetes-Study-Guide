# Using Docker with Minikube

The kubecluster running inside the minikube vm actually uses Docker to run all the containers. So when you create kubernetes objects, e.g. pods, then you can use the docker cli to view the underlying containers that have been created. You might want to do this for troubleshooting/debugging purposes.

In our macbooks, the docker cli is preconfigured to interact with the docker daemon that's running directly on our macbook. At the moment our macbook isn't directly running any containers:

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

However to interact with the minikube's docker daemon, you need to configure your macbook's docker cli to connect to your minikube's docker daemon. That's done by simply setting some environment variables, DOCKER_HOST, DOCKER_CERT_PATH,...etc. The minikube cli helpfully provides these environment variables:

```bash
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="/Users/schowdhury/.minikube/certs"
export DOCKER_API_VERSION="1.35"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

This output also give you a one-liner setup:

```bash
eval $(minikube docker-env)
```

This will only make the changes for the current session and will get reset when you restart are your bash terminal (to make this change permenant you need to add this into your bash profile script files). You should now see something like:

```bash
$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes                                                                             k8s_default-http-backend_default-http-backend-5ff9d456ff-m62k8_kube-system_cee7bc7a-4001-11e9-9566-080027d15c4c_0
da459f1cfb48        quay.io/kubernetes-ingress-controller/nginx-ingress-controller   "/entrypoint.sh /ngi…"   2 minutes ago       Up 2 minutes                                                                             k8s_nginx-ingress-controller_nginx-ingress-controller-7c66d668b-xq5gj_kube-system_cf8d09f1-4001-11e9-9566-080027d15c4c_0
...
```

Notice that we already have some containers running, that's because these containers are used by kubernetes itself for it's internal workings. If you don't have docker installed on your macbook, then you can still use the docker cli that's preinstalled inside the minikube vm. You do that by first ssh'ing into the minikube VM:

```bash
$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)




$ docker container ls
CONTAINER ID        IMAGE                                                            COMMAND                  CREATED             STATUS              PORTS                                                                NAMES
1a62cc23df18        gcr.io/google_containers/defaultbackend                          "/server"                2 minutes ago       Up 2 minutes
...
```

### References

[https://kubernetes.io/docs/setup/minikube/](https://kubernetes.io/docs/setup/minikube/)
[https://kubernetes.io/docs/tutorials/hello-minikube/](https://kubernetes.io/docs/tutorials/hello-minikube/)
[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/tutorials/hello-minikube/)  (talks about getting autocomplete to work)
