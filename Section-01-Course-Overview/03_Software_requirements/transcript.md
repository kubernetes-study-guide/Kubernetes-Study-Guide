Slides -> bash terminal


Now let's take a look at the software you should have on your workstation.

First and foremost you must have the kubectl client installed. This whole course is based on how to use this cli tool. It comes in the form of a single executable binary file that you can simply download and place into one of your path folders.

Next we have minikube. Minikube is one of the official Kubernetes projects and it is used for building a fully functional Kubernetes cluster directly on your local workstation. We'll be using minikube provisioned kube clusters for most of this course.

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


For Windows users you can try using chocolatey instead, chocolatey is like the windows equivalent of homebrew.

Alternatively you can always refer to the official install instructions for each software.


okay lets now go over the workstation hardware requirements. Throughout this course I'll only be using my local workstation to run all my Kubernetes test environments. If you want to do the same, then you need a workstation powerful enough to run minikube. For minikube, your workstation needs to have enough capacity to set aside at least:

- 2 cpu cores
- 2 gigs of RAM
- and 20 gigs of available disk space

If you want to follow along some of our vagrant related demos, you will need set aside at least:

- 6 cpu cores
- 3 gigs of RAM
- 30 gigs of disk space

In case you don't have that, then you can just watch the videos without following along.



Now lets go back to the software requirements. .

In my case I have already installed all the required. So I can't show you homebrew in action. Instead I will run some checks to confirm all the software are definitely installed.

First I'll check virtualbox is installed by running VboxManage version. You can also check this by opening up the virtualbox gui interface.

I'll then perform similar checks for vagrant, docker, minikube, and last but not least Kubectl.

I've included the --short flag to make the output easier to read. By default the kubectl version commnad outputs two versions. The version of the kubectl client binary that's installed locally on your workstation, and the version of the kube cluster that our kubectl is pointing to. So we're using the --client flag here to limit the output to just client side since we don't have a kubecluster yet.

As for VS code, to check if that's installed, it's just a case of seeing if you can open it up.

Ok I'll just clear the screen now. There's one final thing I wanted to do before finishing this video. And that's to enable kubectl's autocomplete feature. To do that we need to run the following source command. After that, you can then try out the autocomplete feature by hitting the tab key twice after your kubectl command. here we can see that kubectl is trying to help us by suggesting what we can write next.

However this approach only lasts for the current bash terminal. So to make it persistant we add it to our .bashrc profile script


Ok we'll take a break here, see you in the next video.




























That's it for this video, see you in the next one.