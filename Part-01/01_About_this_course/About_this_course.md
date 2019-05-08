# Kubernetes By Example

The course takes a learn-by-doing approach so that you get acquainted with as many of the kubernetes features as quickly as possible. At it's core, Kubernetes is all about using the  `kubectl` command along with yaml files that you feed into it. That's why this course is primarily focused on how to write these yaml files and then feeding them into the `kubectl`. 

This course is a follow along hands-on guide to Kubernetes and cover as much of the kubernetes feature as possible. The hope is that you'll understand and learn kubernetes faster by seeing it in action and following along.


## Certified Kubernetes Administrator

In case you're interested, you can get certified by taking the [Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/) exam. This isn't a multiple choice exam, it's actually a hands-on exam where you perform tasks on a live Kubernetes environment. And since this course is a hands-on course, it makes it a good companion guide to help you prepare for the exam.

I should point out that the [exam curriculum](https://github.com/cncf/curriculum) is regularly updated, and this course doesn't quite cover the whole curriculum. However I am planning to create further courses to cover the missing parts in upcoming courses.

In the exam you will be allowed to access all web pages under the [https://kubernetes.io/docs/](https://kubernetes.io/docs/). As well as all pages under [https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes). That's why in this course I will regularly refer to these two sources, so that you can find your around them.

## What's not covered

- Various ways of installing Kubernetes, we only show the minikube approach.
- Other tools that are also used in the Kubernetes ecosystem, e.g. [helm](https://helm.sh/), [traefik](https://traefik.io/),...etc. This course is going to be long enough with just covering Kubernetes on it's own!
- Kubernetes Administration, e.g. updating existing Kubernetes install to a newer version. 
- Cloud platform specific technologies, such as [AWS EKS](https://aws.amazon.com/eks/). This course is going to be cloud agnostic, meaning that what we'll cover should work on most, if not all cloud platforms, whether it is AWS, Azure, GKE...etc.

I'm hoping to cover some of these areas in a future upcoming courses.


## User requirements

Kubernetes is a really powerful software and it's also quite complex. That's why you need to some of the basics before you start learning Kubernetes:

- Linux, Bash, Vim
- Hands-on Docker, so that you understand the concept of containers and images
- yaml files
- git
- basic networking knowledge, e.g. subnets, netmasks, also things like http is port 80.

This isn't a complete list. But it should give you some idea of what level you should be before starting this course. 

## Software requirements

In this course I'll be using a Apple Macbook. So if you're a macbook user and you want to follow along then I recommend that you install the following on your macbook:

- [vscode](https://code.visualstudio.com/)
- [git](https://git-scm.com/downloads) - chances are you already have this installed
- [virtualbox](https://www.virtualbox.org/wiki/Downloads) - Dependency for the minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) software 
- [vagrant](https://www.vagrantup.com/downloads.html) - only needed in a few topics

This above are general purpose and as opposed to Kubernetes specific software. So I'm not going to dwell on how to use them. 

However the following are the main kubernetes software that you need to install. 

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

Argubly the easiest way to install these software on a Mac is with [brew](https://brew.sh/), which itself needs to be installed. Here's an example:

```bash
brew update
brew cask install visual-studio-code
brew install git
brew cask install virtualbox
brew cask install vagrant
brew install kubectl
brew cask install minikube
```

In case you're using a windows laptop, then you'll need to track down and install these software, you can also try using [chocolatey](https://chocolatey.org/) which is the windows equivalent of brew. There are some other softwares that you need to install, we'll cover them later in the course.

In a small number of videos I will be using  Vagrant to run multiple virtual machines on my workstation. So you need to have a have reasonably specced workstation if you want to follow along, ideally 16GB of RAM and a quad core processor. Also about 10 GB of disk available disk storage.

## Study guide

This course comes with it's very own study guide, that has everything you need to follow along, including the example yaml descriptors:

[https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide](https://github.com/Sher-Chowdhury/Kubernetes-Study-Guide)

This study guide is constantly evolving and  is being regularly updated with improvements and new content. Also if you have any suggestions about making changes then please raise an issue or submit a pull request.

## Notations

Throughout this course we'll be using the kubectl command. kubectl is the main command used for performing day-to-day kubernetes work. kubectl has a lot of built in reference docs. That includes lots of man pages:

```bash
man kubectl<tab><tab>
```

Also you can access a lot more info by running:

```bash
kubectl explain xxxxx
```

Where 'xxxxx', is set to something like 'pod.spec'.

The output of some commands are quite long, on those occasions we'll only show an extract using 3-dot notation, and truncate out the rest:

```text
...
output of interest
...
```

## post topic cleanup

After finishing a topic, you should delete everything you created, here's a quick one-liner to do this:

```bash
kubectl delete all --all
```

This command will make more sense once we get started.

## Reference

[https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d](https://towardsdatascience.com/key-kubernetes-concepts-62939f4bc08e?sk=d46386ce56c00701dbf41aa8d308a14d)

[https://github.com/ramitsurana/awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)