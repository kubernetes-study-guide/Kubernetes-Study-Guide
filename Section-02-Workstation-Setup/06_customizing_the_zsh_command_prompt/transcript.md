```
cd Kubernetes-Study-Guide
```


Ok in the last video I mentioned that I'm using the agnoster zed-shell theme. And out of the box this theme's command prompt looks something like this. It shows my username and workstation name, followed by what directory I'm in, which in this case is a directory called Kubernetes-Study-Guide that's under the my home directory. This directory is actually a git repo, which is why the prompt is also showing which git branch I'm on.


As you can see this prompt is quite long and doesn't give a me much room to write my command. So when I try typing something it ends up going to the next line.

```
docker container ls
```

To fix that, I'm going to first try making my prompt shorter by getting rid of the hostname section.

```
hostname
```

That's something I already know so there's no point showing it here as well. I'm also going to shorten the username part as well. At the moment that's showing as

```
whoami
```

So I'll change that to just show my first name, which is only only 4 characters long.


```
echo sher
```

I wasn't sure how to make these changes to the agnoster theme so I had to google for it and worked out that I need to add the following code block to my .zshrc file.


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

Ok that looks better. However I can still run out space if I drill down into several directories. So fix that I'm going to make the command prompt span across 2 lines. That's done by modifying the prompt environment variable:

```
swipe to course notes
```

... and in the course notes you'll find links to websites where you can learn more about this syntax language and build your own command prompt.


But for now, here's one i've prepared earlier.


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

However I'm actually going to use a slightly fancier version of this prompt, which is this one:

```
swipe to browser
```


This prompt is the same as before, except that the arrow turns red if the previous command exits with an error code. This might look quite complicated with all this special zsh prompt syntax, but it's just the arrow characters enclosed in two if-else statements. Where if the exit code of the last command is non-zero, then it outputs the arrow characters highlighted in red otherwise they output as normal.


```
go to http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Conditional-Substrings-in-Prompts
```

. But let

The percentage sign show where this if statement


now lets try this out:

```
copy
swipe to iterm2
vim
```





```
minikube xxx
```

Here we can see arrow head is now red. This is one of the reasons I love zsh.






