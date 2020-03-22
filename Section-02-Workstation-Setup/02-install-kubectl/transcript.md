kubectl comes in the form of a single executable binary file. So you can install it by downloading it into one of your path folders. 

```
echo $PATH
```

Rather than using any of these existing folders. I'm going to create a new personal folder for storing my in. I'll create this folder in my home directory and I'll call it bin, as in short for binary.  

```
mkdir ~/bin
cd ~/bin
```


Now let's take a look at the official kubectl install guide.  
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos


Here it gives the curl command to download the file. So let's turn that:

```
curl 
ll
```

Ok that's downloaded, but I need to make it execuble, so let's do that now. 


Now I need to add my new bin folder to the PATH variable. I'll do that by append the following line to my shell profile script. 

echo "export $PATH=~/bin:$PATH" >> ~/.zshrc


I'm using zsh as my shell, which is why I'm appending to the zshrc file. however if you're using bash, then see the course notes for the bash equivalent.

I need to reload my shell to load in the new PATH variable. 

```
echo $PATH 
```

here we can see the new bin folder is listed here. Let's confirm that my shell session can now locate the kubectl binary, using the which utility. 

```
which kubectl 
```

so far so good. Now let's try using kubectl. 

```
kubectl version --client 
```

That looks like it has worked. The version command also gives the version of the kubernetes cluster that it's currently pointing to. But since we don't have a kubernetes cluster yet, it means taht this command would just hang waiting for a response.  That's why I've used the client flag to suppress kubectl from trying to reach out to any kube clusters.


This output is a bit difficult to read, which is why I like to use the  --short flag to make it more human readable:


```
kubectl version --client --short
```

If there are any other single executable binary files I want to install in future, Then I can now just curl it into this new personal bin folder. This approach helps me to keep track of which tools I've installed manually using curl, and which tools have been installed using a package manager, such as homebrew. By the way I find using package managers a bit of an overkill for installing single file binaries, which is why I've installed kubectl manually.  



We'll be writing a lot of kubectl commands in this course. To help with I'm going enable the kubectl autocomplete feature. That's done by appending the following line to muy shell's profile script. 

```
echo "source <(kubectl completion zsh)" >>~/.zshrc
```

And then restart my shell to load in this change. Now we can type kubectl and hit tab twice to get a list of the available sub commands. 