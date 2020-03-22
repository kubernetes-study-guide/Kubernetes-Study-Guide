Ok let's install minikube. as I'm a mac user, i'm going to install it using homebrew:

```
brew install minikube
```

Alright that's seems to have installed successfully. Let's now test it by running minikube version:


```
minikube version
```



Ok that didn't go as a planned. To be honest I was half expecting this because I've seen this error before. To fix it all I had to do was run the brew-link command.

```
brew link minikube
```


```
minikube version
```

Cool Now let's check that status:

```
minikube status
```

That looks better. As expected it's saying that we haven't created our minikube provisioned kube cluster yet.

Let's try to create it now.

```
minikube start
```

ok this time it failed, because minikube couldn't find a hypervisor to use to spin up a vm with.

I can install a hypervisor such as virtualbox. However docker comes with a builtin hypervisor, so let's put minikube on the back burner for now and install docker.

I've already opened up the docker download page in google chrome so lets start downloading it. This will take a few minutes so I'm fast forward the video.


ok that's now downloaded. Let's try installing it.


It's asking me to drag docker into my apps folder. So let's do that.

It's looks like I may have had an older version of docker left behind from a previous install. So I'm just gonna replace that.

Ok the install has finished so let's try launching it.


And now enter my password.


Docker is now starting up. I won't bother logging at the moment, i'll do that later if-and-when I need to.

Ok got confirmation that docker is now up and running. Let's go back to minikube and try starting that up again.

```
minikube start
```

this time minikube has found the hyperkit hypervisor which came with docker and now Minikube is asking to run a couple of sudo commands which is why it's asking for my password.

it's now downloading some iso files, and then its creating a vm and it's allocating 2 cpu cores from my workstations, along with 4 gigs or memory and 20 gigs of disk space. NOw its install all the kubernetes components onto that vm which effectively will result in a single node kube cluster.

Ok it looks like it's finished building the kube cluster so let's check the status again:

```
minikube status
```



So I now have a kubecluster that's running locally on my macbook.

Minikube has also taken care of configuring our kubectl client to point to this local kube cluster. Let's test this by running kubectl version:

```
kubectl version --short
```

This time we can see both the client and server versions. They are slight different but that should be a problem.

Let's now check that our kubectl is definitely pointing to this minikube cluster. To do that first let's get the cluster-info:

```
kubectl cluster-info
```

Here it show that our kubecluster's ip address has this 192 ip address. Now let's compare that to our minikube vm's ip address:

```
minikube ip
```

As you can it matches, which means that kubectl is definitely talking to minikube provisioned cluster.