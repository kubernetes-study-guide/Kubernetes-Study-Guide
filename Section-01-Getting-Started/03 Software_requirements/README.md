# Course Requirements

This course has 3 sets of requirements:

- User Requirements
- Software Requirements
- Hardware Requirements

## User requirements

Kubernetes is a really powerful software and it's also quite complex. That's why you need to some of the basics before you start learning Kubernetes:

- **Linux** (e.g. CentOS or Ubuntu), Bash, Vim
- **Hands-on Docker experience** - that's so that you understand the concept of containers and images
- **Basic networking knowledge** -  Such as IP addresses and port numbers, also you need to know about commands like netcat, curl, dig, nslookup..etc.

This isn't a complete list. But it should give you some idea of the level you should be at before starting this course.

## Software Requirements

In this course I'll be using a Apple Macbook. So if you're a macbook user and you want to follow along then I recommend that you install the following on your macbook:

- [VS Code](https://code.visualstudio.com/) - Text editor for writing our code. 
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) - Used for building Virtual Machines on your local workstation
- [vagrant](https://www.vagrantup.com/downloads.html) - Used for spinning 1 or more virtual machines by running a single command
- [docker](https://www.docker.com/get-started)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - The main tool for interacting with Kubernetes clusters
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) - Used for building a Kubernetes cluster right on your local workstation

Most of these are general purpose tools used for software development. The only exception being the last 2, kubectl and minikube, these 2 are Kubernetes specific.

Probably the easiest way to install these software on a Mac is with [brew](https://brew.sh/), which itself needs to be installed. Here's an example:

```bash
brew cask install visual-studio-code
brew cask install virtualbox
brew cask install vagrant
brew install docker
brew install kubectl
brew cask install minikube
```

In case you're using a windows laptop, then you'll need to track down and install these software, you can also try using [chocolatey](https://chocolatey.org/) which is the windows equivalent of brew. There are some other softwares that you need to install, we'll cover them later in the course.

## Workstation Hardware Requirements

In a small number of videos I will be using Vagrant to run multiple virtual machines on my workstation. So you need to have a have reasonably specced workstation if you want to follow along, ideally 16GB of RAM and a quad core processor. Also about 10 GB of disk available disk storage. In case you don't have that then you can skip and just the videos instead.


## Post install tasks

After installing the software listed above, you should do a few quick checks to confirm you now have everything installed, we'll do this by performing version checks:

```bash
$ VBoxManage --version
6.0.4r128413

$ vagrant --version
Vagrant 2.2.4

$ docker --version
Docker version 18.09.2, build 6247962

$ minikube version
minikube version: v1.0.1

$ kubectl version --client --short
Client Version: v1.14.1
```

Next you need to enable [kubectl's autocomplete](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete) feature:

```bash
source <(kubectl completion bash) # setup autocomplete in bash into the current shell
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

CKA tip: It's really important to do this as the very first thing in the exam. 

For VScode, I'll be using a couple of extensions, which I recommend you also install:
- [installed code cli for VS code](https://code.visualstudio.com/docs/setup/mac)
- [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager) - Then save the Kubernetes Study Guide as a project. This acts as a bookmark, that lets you open up the Study Guide with just a couple of click
- [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) - This is useful for syntax checking yaml files
- [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) - This displays README.md files in a nice readable format.




