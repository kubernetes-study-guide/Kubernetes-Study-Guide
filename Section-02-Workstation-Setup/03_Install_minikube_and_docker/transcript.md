Ok as I'm a mac user, so i'm going to install minikube using homebrew:

```
brew install minikube
```

Alright that's seems to have installed successfully. Let's now test it by running minikube version:


```
minikube version
```



Ok that didn't go as a planned. To be honest I was half expecting this because I've seen this error in the past. To fix it all I had to do was run the brew-link command.

```
brew link minikube
minikube version
```

Now let's check that status:

```
minikube status
```

That looks better. As expected it's saying that we haven't created our minikube provisioned kube cluster yet.

Let's try to create it now.

```
minikube start
```

ok this time it failed failed because minikube couldn't find a hypervisor to use to spin up a vm with.

Now i can install a hypervisor now such as virtualbox. However docker comes with a builtin hypervisor, so let's put minikube on the back burner for now and install docker.

I've already opened up the docker download page in another window so let's start downloading it. This will take a few minutes so I'm fast forward the video.


ok that's now downloaded. Let's try installing it.


Ok looks like i have to do a bit of dragging and dropping.


It's looks like I may have had an older version of docker left behind from a previous install. So I'm just gonna replace that.

Ok it looks like the install has finished so let's try launching it.


And now enter my password.


Cool it looks like it's starting up now. I won't bother logging now I'll do that later.

Ok got confirmation that docker is now up and running. Let's go back to minikube and try starting that up again.

Cool this time it looks like we're making more progress. Minikube needs to run a couple of sudo commands which is why it's asking for my password.

Now it looks like it's downloading some files, and then its creating a vm and it's allocating 2 of workstations cpu cores, along with 4 gigs or memory and 20 gigs of disk space. NOw its install all the kubernetes components onto that vm which effectively will result in a single node kube cluster.

Ok it looks like it's finished building the kube cluster so let's check the status again:

```
minikube status
```



So I now have a kubecluster that's running locally on my macbook.

Minikube has also taken care of configuring our kubectl client to point to this local kube cluster. Let's test this by running kubectl version:

```
kubectl version --short
```

This time we can see both the client and server version. They are slight different version but that shouldn't cause any problems.

Let's now check that our kubectl is definitely pointing to this minikube cluster. To do that first let's get the cluster-info:

```
kubectl cluster-info
```

Here it show that our kubecluster's ip address has this 192 ip address. Now let's compare that to our minikube vm's ip address:

```
minikube ip
```

As you can it matches, which means that kubectl is definitely talking to minikube provisioned test cluster.