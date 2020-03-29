Ok in the last vidoe I mentioned that I'm using the agnoster zed-shell theme. And out of the box this theme's command prompt looks something like this. It shows my username and workstation name, followed by what directory I'm in.

So there's a small problem with this to see what I mean I'll cd into a folder,

```
cd Kubernetes-Study-Guide
```

now my prompt is a bit longer and doesn't give a me with a lot of room to write my command, without it going into the next line.

```
docker container ls
```

To get past that, I'm going to modify my prompt to make it shorter. First I'll get rid of the workstation name since I already know what that is, and I'll set my name to just my first name, which is shorter. To do that I'm going to go into my .zshrc file and add the following at the end.


```
swipe
copy and paste from the block from the course notes.
```

Now restart the shell


```
vim save, then restart the shell
```

Ok that looks better. To make it even better I'm going to make the command prompt span across 2 lines. That's done by modifying the prompt environment variable:

```
echo $PROMPT
```



To change this I need to set the new PROMPT setting in the zshell config file.


```
vim zshrc # scroll to bottom
```

In the course notes you'll find links to places about customising the zsh command prompt. However in my case, here's one I've prepared earlier so I'll copy and paste this into my zsh profile:

```
swipe to course notes
```

Now lets restart my shell and see what it looks like.


```
restart shell
```

That looks a lot better. Now I've got the whole width of the terminal.


```
docker container ls
```

I've also added a small arrow on the left to show that this is a single command prompt that spans across 2 lines.

This prompt also has a bonus feature which is that this arrow head turns red if the previous command exits with an error code:

```
minikube xxx
```

Here we can see arrow head is now red. This is one of the reasons I love zsh.






