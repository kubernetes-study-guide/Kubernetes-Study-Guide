# Kubernetes By Examples

The course takes a learn-by-doing approach so that you get acquainted with as many of the kubernetes features as quickly as possible. About 80% of day-to-day kubernetes work involces `kubectl` command.

## The aim of this course

This course is a follow along hands-on guide to Kubernetes and cover as much of the kubernetes feature as possible, so that you get comfortable using kubernetes for usual day-to-day work. This course is also a good companion guide for those who are planning on taking the [Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/) certification.

This course is also heavily focused helping you get familiar with the official kubernetes documentation, so that you know your way around it. Since you will have access to the docs during the exam.

## Requirements

This is an intermediate level course. You need to know the following:

- docker and concept of containers
- git
- linux, bash, vim
- yaml syntax
- basic networking knowledge, e.g. subnets, netmasks, also things like http is port 80.

## Software requirements

In this course I'll be using a Apple Macbook. So if you're a macbook user and you want to follow along then I recommend that you install the following on your macbook:


- [git](https://git-scm.com/downloads)
- [virtualbox](https://www.virtualbox.org/wiki/Downloads)


 Argubly the easiest way to install these sofware is with [brew](https://brew.sh/), which itself needs to be installed. Here's an example:

```bash
brew update
brew cask install visual-studio-code
brew install git
brew cask install virtualbox
brew cask install vagrant
```

In case you're using a windows laptop, then you'll need to track down and install these software if you don't already have them, you can also try using [chocolatey](https://chocolatey.org/) which is the windows equivalent of brew. 

There are some other softwares that you need to install, we'll cover them later in the course.

As part of this course we'll be running 

## scope

We don't spend that much time going over concepts and theories. The hope is that you'll understand and learn kubernetes faster by seeing it in action.
- Kubernetes installation - only focusing no minikube on macs
- docker, this course won't cover a lot about docker

## Study guide

This course comes with it's very own study guide, that has everything you need to follow along.

most of the demos are done on a minikube based kubecluster. But some demo requires a multi-node cluster so we used a

## Notations

Throughout this course we'll be using the kubectl command. kubectl is the main command used for performing day-to-day kubernetes work. kubectl has a lot of built in reference docs. That includes lots of man pages:

```bash
man kubectl<tab><tab>
```

Also you can access a lot more info by running:

```bash
kubectl explain xxxxx
```

In this case, we'll refer to what to put in here, using a dot like notation e.g. 'pod.spec'.

Also some commands have a long output, so we only show an extract inside 3-dot notation:

```text
...
output of interest
...
```

We'll be using shorthands where we can, to cut down typing:

```bash
$ kubectl get deployments,pods
NAME          READY   STATUS    RESTARTS   AGE
pod/podname   1/1     Running   0          10d
```

## post topic cleanup

After finishing a topic, you should delete everything you created, here's a quick one-liner to do this:

```bash
kubectl delete all --all
```

Note: don't worry if it deletes the kubernetes service, that will just get recreated again.

## Reference

[https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d)

[https://github.com/ramitsurana/awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)