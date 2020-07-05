kubectl comes in the form of a single executable binary file. So you can install it by downloading it into one of your path folders.

```
echo $PATH
```

Rather than using any of these existing folders. I'm going to create a new personal folder for storing my executables in. I'll create this folder in my home directory and I'll call it bin, as in short for binary.

```
mkdir ~/bin
cd ~/bin
```


Now let's take a look at the official kubectl install guide.
https://kubernetes.io/docs/tasks/tools/install-kubectl/

I've already opened up the web page for it, so I just do jump to the macos section.


Here it gives the curl command to download the file. So let's run that:

```
curl
ll
```

Ok that's downloaded, next I need to make this file executable.



```
chmod +x ./kubectl
```

Now I need to add my new bin folder to the PATH variable. I'll do that by appending the following line to my shell profile script.

```
echo "export PATH=~/bin:$PATH" >> ~/.zshrc
```

I'm using zsh as my shell, which is why I'm sending this line to the zshrc file. however if you're a bash user, then see the course notes for the bash version of this command.

This PATH setting will take effect from the next restart. So let's restart my terminal to load this in.

```
echo $PATH
```

here we can see our new binary folder is listed here. Let's confirm that my shell can now locate the kubectl binary, using the which utility.

```
which kubectl
```

so far so good. Now let's try actually using kubectl by getting it to print out it's version.

```
kubectl version --client
```

Cool it looks like it's working. The version command also gives the version of the kubernetes cluster that kubectl is currently configured to point to. But since we don't have a kube cluster yet, it means taht this command would just hang waiting for a response.  That's why I've used the client flag to limit kubectl to just printing out the client version.


This output is a a little hard to make out, which is why I like to use the  --short flag to make it more human readable:


```
kubectl version --client --short
```

If there are any other single executable binary files I want to install in future, Then I can now just curl it into my new personal binary folder. This approach helps me to keep track of which tools I've installed manually using curl, and which tools have been installed using a package manager, such as homebrew. By the way I find using package managers a bit of an overkill for installing single file binaries, the why I installed kubectl manually.



One final thing I wanted to show you is how to enable the kubectl autocomplete feature so that kubectl helps you to write your commands faster. All you have to do is append this line to your shell's profile script.

```
echo "source <(kubectl completion zsh)" >>~/.zshrc
```

And then restart your shell to load in this change. Now when we type kubectl followed by the tab key, kubectl lists out the available sub commands to choose from.
