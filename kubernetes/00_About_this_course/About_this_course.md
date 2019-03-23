## Possible Kubernetes Titles

- kubernetes deep dive
- Kubernete - Hands on
- Kubernetes in Action
- Kubernetes by example



## The aim of this course

This course is a follow-along hands-on guide to Kubernetes and cover as much of the kubernetes feature as possible, so that you get comfortable using kubernetes for usual day-to-day work. 

This course is also a good companion for those who are planning on taking the [Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/) certification.


## Requirements

This is an intermediate level course. You need to know the following:

- docker and concept of containers
- git
- linux, bash, vim
- yaml syntax
- basic networking knowledge, e.g. subnets, netmasks, also things like http is port 80. 
- already have basic kubernetes knowledge - This isn't a kubernetes beginners course. 


## what software you need to follow along

- vscode
- brew
- git
- virtualbox 



## scope

- We don't spend that much time going over concepts and theories. The hope is that you'll understand and learn kubernetes faster by seeing it in action. 
- Kubernetes installation - only focusing no minikube on macs
- docker, this won't cover a lot about docker
- 

## Study guide

This course comes with it's very own study guide, so you can follow along all the examples. 

This course will be closely following the following git repo: xxxxxx

most of the demos are done on a minikube based kubecluster. But some demo requires a multi-node cluster so we used a


# Course Structure 

How to follow this course


## Notations.

Throughout this course we'll be using the kubectl command. kubectl is the main command used for performing day-to-day kubernetes work. kubectl has a lot of built in reference docs. That includes lots of man pages:

```bash
man kubectl<tab><tab>
```

Also you can access a lot more info by running:

```bash
kubectl explain xxxxx
```

In this qude, we'll refer to what to put in here, using a dot like notation e.g. 'pod.spec'. 


Also some commands have a long output, so we only show an extract inside 3-dot notation:

```bash
...
output of interest
...
```

We'll be using shorthands where we can, to cut down typing:

```bash
 kubectl get deployments,pods
```

## post topic cleanup

After finishing a topic, you should delete everything you created, here's a quick one-liner to do this:


```bash
$ kubectl delete all --all
```

Note: don't worry if it deletes the kubernetes service, that will just get recreated again.

## Disclaimer

I have not taken my KCA exam yet. 