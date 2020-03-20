kubectl comes in the form of a single executable binary file. So to install it you can just download that file into one of your path folders. Just to keep things organised so I know which binaries I've downloaded as opposed to binaries have installed by a package manager like brew or apt, I'm going to create a new folder for storing my downloaded binaryies. I'll create this folder in my home directory and I'll call it bin, as in short for binary.  


mkdir ~/bin
cd ~/bin

Next I'll run the curl command to download the binary. you can find this curl command in the installation guide:

https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos



```
curl 
ll
```

Ok that's downloaded, but I need to make it execuble, so let's do that now. 



Now the final step is to add this folder to my PATH variable. and append this to my profile script. 

echo "export $PATH=~/bin:$PATH" >> ~/.zshrc


I'm using zsh as my shell, however if your using bash, then see the course notes for the bash equivalent. 

Now I'll need to restart my shell to load in the new PATH variable. Le'ts now chech that my shell session can now locate the kubectl binary. 

```
which kubectl 
```

so far so good. Now let's try using kubectl. 

```
kubectl version --client 
```

That looks like it has worked. Although it's a bit difficult to read, luckily the version subcommand comes with a --short flag to get round that:


```
kubectl version --client --short
```

The kubectl command by default gives two version info, One for the kubectl client and the other for the kubernetes instance it's configured to talk to. Since we don't have a kubernetes cluster yet it would cause this command would hang. That's why I've used the client flat to instruct kubectl to only give the client version.  

Kubectl comes with an autocomlete feature that will be a massive help to write out kubectl commands faster. To enable autocomplet
you need to run:


echo "source <(kubectl completion zsh)" >>~/.zshrc


Once again, in my case I'm using zsh, see the course notes for the bash version of this command. 
