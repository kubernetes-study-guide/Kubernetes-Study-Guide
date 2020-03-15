As for the shell itself, I'll be using zsh instead of bash.

```
echo $SHELL
```


zsh is basically the same as bash, but with a lot more feature. In 2019 zsh replaced bash to become the default shell in  MacOS. So if you're tempted in switching to zsh, then now would be a great time to jump ship. However all the commands I'll demo will work fine on both shells.

In my case not only have switched to using zsh, I've also turbocharged it installing the oh-my-zsh framework. This framework gives me access to a lot of extra features  including a bunch of handy aliases:

```
alias
```

For example if I want to run "kubectl get pods", then I can just use the kgp alias followed by space:

```
kgp<space>
```

Hitting space ends up replacing the alias with the actual underlying command it referenced to.

Pretty cool right. I'll be using these aliases a lot to cut down my typing and to get things done quicker.

By the way, these aliases are only available after activatin some om-my-zsh plugins. To do that I had to go to my home directory and open up the zsh config file, which is called .zshrc. In this file there's a section where you can select your plugins.

So if you're already a zsh user but haven't used this framework before, then I definitely recommend giving it a try. It's dead easy to install and you can find the install instructions in this video's description.


As for my code editor, I'll be using VS Code. VS Code has it's own integrated terminal. You might have noticed that my VS Code interface looks a little different to what you might be expecting, and that's because I'm using the Cobalt2 vscode theme. Check out this video's description for a link to that theme  .

That's it for this video, see you in the next one.




Slides -> bash terminal

- watch
- zsh (oh my zsh framework) along with plugins. 
- zsh-syntax-highlighting
- iterm2
- [VS Code](https://code.visualstudio.com/) - Text editor for writing our code. themes and plugins
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) - Used for building Virtual Machines on your local workstation
- [docker](https://www.docker.com/get-started)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - The main tool for interacting with Kubernetes clusters
- [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) - Used for running Kubernetes locally on your workstation


Now let's take a look at the software you should have on your workstation.

First and foremost you must have the kubectl client installed. This whole course is based on how to use this cli tool. It comes in the form of a single executable binary file that you can simply download and place into one of your path folders.

Next we have minikube. Minikube is one of the official Kubernetes projects and it is used for building a fully functional Kubernetes cluster directly on your local workstation. We'll be using minikube provisioned kube clusters for most of this course.

Minikube needs to make use of a hyper visor to build a VM, and for that we'll be using Virtualbox.

We also have a few optional extras that you may like to install as well:






The quickest way to install all of these software is probably by using a tool called homebrew, This cli tool lets you install all these applications by just running a few commands.


For Windows users you can try using chocolatey instead, chocolatey is like the windows equivalent of homebrew.

Alternatively you can always refer to the official install instructions for each software.




Now lets go back to the software requirements.

In my case I have already installed all the required. So I can't show you homebrew in action. Instead I will run some checks to confirm all the software are definitely installed.

First I'll check virtualbox is installed by running VboxManage version. You can also check this by opening up the virtualbox gui interface.

I'll then perform similar checks for docker, minikube, and last but not least Kubectl.

I've included the --short flag to make the output easier to read. By default the kubectl version commnad outputs two versions. The version of the kubectl client binary that's installed locally on your workstation, and the version of the kube cluster that our kubectl is pointing to. So we're using the --client flag here to limit the output to just client side since we don't have a kubecluster yet.

As for VS code, to check if that's installed, it's just a case of seeing if you can open it up.

Ok I'll just clear the screen now. There's one final thing I wanted to do before finishing this video. And that's to enable kubectl's autocomplete feature. To do that we need to run the following source command. After that, you can then try out the autocomplete feature by hitting the tab key twice after your kubectl command. here we can see that kubectl is trying to help us by suggesting what we can write next.

However this approach only lasts for the current bash terminal. So to make it persistant we add it to our .bashrc profile script


Ok we'll take a break here, see you in the next video.




























That's it for this video, see you in the next one.