# Requirements

## User requirements

Kubernetes is a really powerful software and it's also quite complex. That's why you need to some of the basics before you start learning Kubernetes:

- Linux, Bash, Vim
- Hands-on Docker experience, so that you understand the concept of containers and images
- yaml files
- basic networking knowledge, e.g. subnets, netmasks, also things like http is port 80.

This isn't a complete list. But it should give you some idea of the level you should be at before starting this course.

## Software requirements

In this course I'll be using a Apple Macbook. So if you're a macbook user and you want to follow along then I recommend that you install the following on your macbook:

- [vscode](https://code.visualstudio.com/)
- [git](https://git-scm.com/downloads) - chances are you already have this installed
- [virtualbox](https://www.virtualbox.org/wiki/Downloads) - Dependency for the minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) software
- [vagrant](https://www.vagrantup.com/downloads.html) - only needed in a few topics
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

Most of these are general purpose tools used for software development. The only exception being the last 2, kubectl and minikube, these 2 are Kubernetes specific.

Probably the easiest way to install these software on a Mac is with [brew](https://brew.sh/), which itself needs to be installed. Here's an example:

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

## Workstation harware requirements

In a small number of videos I will be using  Vagrant to run multiple virtual machines on my workstation. So you need to have a have reasonably specced workstation if you want to follow along, ideally 16GB of RAM and a quad core processor. Also about 10 GB of disk available disk storage.
