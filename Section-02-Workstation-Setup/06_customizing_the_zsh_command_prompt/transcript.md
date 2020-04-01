<make iterm2 fon a bit bigger and split terminal into two vertial terminals>

```
cd Kubernetes-Study-Guide
```


Ok in the last video I mentioned that I'm using the agnoster zed-shell theme. And out of the box this theme's command prompt looks something like this. It shows my username and workstation name, followed by the directory path. This folder happens to be a git repo, and that's why the prompt also shows the currently active git branch.

```
git status
```


this prompt is quite long and doesn't give me much room to write my command especially in a split terminal setup like this. That's why my git-status command ended up going to the next line.


To fix that, I'm going to free up some room by getting rid of the hostname section.

```
hostname
```

That's something I already know so there's no point showing it here as well. I'm also going to shorten the username bit. At the moment that's showing my initial and lastname

```
whoami
```

So I'll change that to just show my first name, which is only only 4 characters long.


```
echo sher
```

I wasn't sure how to make these changes to the agnoster theme so I had to google for it. And I eventually worked out that I needed to add the following code block to my .zshrc file.


```
swipe to course notes
copy and paste from the block from the course notes.
swipe to iterm2
vim ~/.zshrc
scroll to the bottom
paste
save
```

So let's do that now.

ok I now need to restart the shell for this change to take effect.


```
restart the shell
```

Ok that looks better. However I can still run out space if I drill down into several directories.

```
cd Section-02
```


So fix that I'm going to make the command prompt span across 2 lines. That's done by modifying the prompt environment variable:

```
swipe to course notes
```

and Here's the one I'm going to use. Once again I have to add this to my zshell profile:

```
swipe to iterm
vim ~/.zshrc
vim zshrc # scroll to bottom
paste
then save
```

Now let's restart the shell to load this in:

```
restart the shell
```

Cool, it looks like that did the trick. Now I've got the whole width of the terminal to play with.

```
git checkout -b test
```

I've also added a small arrow on the left. This is more for cosmetic purposes, just to show that this is a single command prompt that spans across 2 lines.



However I'm actually going to use a slightly fancier version of this prompt, which is this one:

```
swipe to browser
```


This prompt is the same as before, except that the arrow turns red if the previous command exits with an error code. This might look quite complicated with all this special zsh prompt syntax, but it's actually just a couple of if-else statements. Where if the exit code of the last command is non-zero, then it outputs the arrow with red highlighting, otherwise they output as normal.


So to use this I need replace my existing prompt setting with this one, so let's do that now:

```
copy
swipe to iterm2
vim
restart shell
```

Now let's try this out by running an bad command



```
git xxx
```

Awesome the arrow is now red. As an added bonus, the agnoster theme is also showing a red cross to indicate something has gone wrong. And when I run a valid command it goes back to normal:

```
git status
```





