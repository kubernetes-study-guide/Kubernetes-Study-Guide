```
cd Kubernetes-Study-Guide
```


Ok in the last video I mentioned that I'm using the agnoster zed-shell

theme. And out of the box this theme's command prompt looks something like this. It shows my username and workstation name, followed by what directory I'm in, which in this case is a directory called Kubernetes-Study-Guide which is under my home directory. This directory is actually a git repo, which is why the prompt is also showing which git branch I'm on.


However this also means that this prompt is quite long and doesn't give a me a lot of room to write my command. So When I try to type out a command it ends up going to the next line.

```
docker container ls
```

To fix that, I'm going to modify my prompt to make it shorter. First I'll get rid of the workstation name since I already know what that is, and I'll set my name to just my first name. I wasn't sure how to do for the agnoster theme so I had to google for it and worked out that I need to add the following code block to my .zshrc file.


```
swipe to course notes
copy and paste from the block from the course notes.
swipe to iterm2
vim ~/.zshrc
scroll to the bottom
paste
save
```

Now I need to restart the shell for this change to take effect.


```
restart the shell
```

Ok that looks better. However I can still run out space if I drill down into several directories. So fix that I need to make some more general customisations to the command prompt. This time I'm going to make the command prompt span across 2 lines. That's done by modifying the prompt environment variable:

```
echo $PROMPT
```

zsh has it's own syntax for configuring this prompt variable, and I've include links to website where you can learn more about this syntax language and build your own command prompt.

But here's one i've prepared earlier.

```
swipe to course notes
```

Once again I have to add this to my zshell profile:

```
swipe to iterm
vim ~/.zshrc
vim zshrc # scroll to bottom
paste
then save
```

Now let's load this in by restarting the shell:

```
restart the shell
```

Cool, it looks like that did the trick. Now I've got the whole width of the terminal for writing my commands.
I've also added a small arrow on the left to show that this is a single command prompt that spans across 2 lines.

```
docker container ls
```



This prompt also has a bonus feature which is that this arrow head turns red if the previous command exits with an error code:

```
minikube xxx
```

Here we can see arrow head is now red. This is one of the reasons I love zsh.






