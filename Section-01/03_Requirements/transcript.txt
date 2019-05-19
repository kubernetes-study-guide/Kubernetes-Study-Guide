Hello Everyone and Welcome back.  

Ok, So what are the requirements for this course to get the most out of it? 


there are three sets of requirements


 - user requirements
 - software requirements 
 - and hardware requirements 

Let's take a look at user requirements first. 

This course is aimed at people who already know their way around Linux. It doesn't matter which distro in particular but I'll be using CentOS and Ubuntu. You should be comfortable using a bash terminal and editing files with VIM. 


experience with Docker is important as well, so that you understand the concept of containers and images and how they are used. 


You should have some basic networking knowledge so that you're familiar with things like IP addresses and port numbers.

You also need to know how to use various network troubleshooting commands like netcat, curl, dig, nslookup,...and etc. 


This should give you a general idea of the level you should be at before starting this course.

 
 
Now let's take a look at the software you should have on your workstation. 

First and foremost you must have the kubectl client installed. This whole course is based on how to use this cli tool. It comes in the form of a single executable binary file that you can simply download and place into one of your path folders.

Next we have minikube. Minikube is one of the official Kubernetes projects and is used for building a fully functional Kubernetes cluster directly on your local workstation. We'll be using minikube provisioned kube clusters for most of this course. 

Minikube needs to make use of a hyper visor to build a VM, and for that we'll be using Virtualbox. 

We also have a few optional extras that you may like to install as well:

We'll be using VS Code as our text editor. 
Although you can use whatever text editor that you prefer. 
We've also installed a VS code extension called 
"Markdown Preview Enhanced",
 which lets you view README files in a human readable format. 
 On top of that I have installed vs code's own cli tool, which is called 'code'. 

In some of our demos we'll need to make use of a multinode kube cluster. On those occasions we'll use vagrant to build them. 


The quickest way to install all of these software is probably by using a tool called homebrew, This cli tool lets you install all these applications by just running a few commands. 
  
  
For Windows user you can try using chocolatey instead, chocolatey is like the windows equivalent of homebrew. 

Alternatively you can always refer to the official install instructions for each software. 
 

okay lets now go over the workstation hardware requirements. Throughout this course I'll only be using my Macbook to run all my Kubernetes test environments. If you want to do the same, then you need a workstation powerful to use minikube. For minikube, you need to set aside at least:
 
- 2 cpu cores
- 2 gigs of RAM
- and 20 gigs of available disk space

Whereas for a vagrant provisioned multi-node kubecluster, you will need set aside at least:

- 6 cpu cores
- 3 gigs of RAM
- 30 gigs of available disk space

In case you don't have that, then you can just watch the videos without following along.





In my case I have already installed all the required apps, so I won't be able to demonstrate homebrew. Instead I will confirm all the installs have been successful by running a series of version check commands.

First I'll check virtualbox is installed by checking the VboxManage version.

I'll then perform similar checks for vagrant, docker, minikube, and last but not least Kubectl. 

I've included the --short flag to make the output easier to read. By default the kubectl version outputs two versions. The version of the local kubectl client binary that's installed on your workstation, and the version of the kube cluster that our kubectl is pointing to. So we're using the --client flag to limit the output to just client side since we don't have a kubecluster yet. 

As for VS code, to check if that's installed, it's just a case of seeing if you can open it up. 

Ok we'll take a break here, see you in the next video. 




























That's it for this video, see you in the next one. 