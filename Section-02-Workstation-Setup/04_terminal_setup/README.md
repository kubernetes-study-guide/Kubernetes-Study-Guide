# Course Requirements

In case you're using a windows laptop, then you'll need to track down and install these software, you can also try using [chocolatey](https://chocolatey.org/) which is the windows equivalent of brew. There are some other softwares that you need to install, we'll cover them later in the course.

In this course I'll be using a Apple Macbook. So if you're a macbook user and you want to follow along then I recommend that you install the following on your macbook:

- watch
- zsh (oh my zsh framework) along with plugins. 
- iterm2
- [VS Code](https://code.visualstudio.com/) - Text editor for writing our code. 
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) - Used for building Virtual Machines on your local workstation
- [docker](https://www.docker.com/get-started)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - The main tool for interacting with Kubernetes clusters
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) - Used for running Kubernetes locally on your workstation

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



