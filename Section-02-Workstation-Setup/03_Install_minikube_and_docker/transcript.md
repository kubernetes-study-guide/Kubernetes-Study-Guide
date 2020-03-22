Ok as I'm a mac user, so i'm going to install minikube using the homebrew package manager:

```
brew install minikube
```

Ok that's seems to have installed ok. Let's now test it by running minikube version:


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

As expected it's saying that we haven't created our minikube provisioned kube cluster yet.

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

Now it looks like it's downloading some files, and then its creating a vm and it's allocating 2 of workstations cpu cores, along with 4 gigs or memory and 20 gigs of disk space. NOw its install all the kubernetes components onto that vm which effectively will result in a single node cluster.

Ok it looks like it's finished building the cluster and it has configured our kubectl client to point to it.

let's check the status of our minikube clsuter:

```
minikube status
```

As you can see, I now have a kubecluster that's running locally on my macbook. We can now run kubectl commands against this local cluster:

```
kubectl version --short
```