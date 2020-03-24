Alright. Let's take a look at my terminal setup.

```
click rocket icon - then search for 'term'
```


In this course I won't be using my macos's builtin terminal. Instead I've installed a popular alternative called iterm2. It shows iterm on the laaunch screen here, but it actually called iterm2 after you open it up.

```
open terminal -> hover over the header
```


iterm2 has a lot of cool capabilites and customization options, one of my favourite feature is that it lets me easily split my terminal into smaller terminals.

```
do vertical and horizontal splits
```

So if you're a mac user then you might like to give iterm2 a try.

Now let's talk about my shell setup.

For my shell I'll be using zsh.

```
echo $SHELL
```

zsh is like bash but with a lot more features. In the year 2019 Apple made zsh the default shell in macos, before it used to be bash. So if you're thinking of making the switch to zsh then now would be a great time to jump ship. In my experience nearly everything I used to do on bash, works just as well in zsh, if not better.

I've also made zsh even better by installing the oh-my-zsh framework.

```
swipe to web browser - tab1 = https://github.com/ohmyzsh/ohmyzsh
```

It's dead easy to install, all you have to do is run a curl command.

```
scroll to curl command
```



This framework gives you access to a lot of cool themes.

```
tab2 - https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
```


There's also a long list of zsh plugins you can make use of as well:

```
tab3 - https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins
```

You can set your themes and plugins by by going into your .zshrc file.


```
swipe
vim ~/.zshrc
```


In my case I've activated the gianu theme.


I've also activated several plugins by inserting their name in the plugins section.

Let's go over some of these plugins

The docker plugin simply enables the docker autocomplete feature.

````
vertical split, vim in right terminal
````




The kubectl plugin does 2 things. First it another way of enableing the kubectl autocomplete and secondly it also sets up bunch of handy aliases.

```
aliases | grep kubectl
```


These aliases are a massive time saver but can appear a bit cryptic if you're familiar. To solve that problem I've activated the globalias plugin. This auto expands an alias to the actual when I hit space after it. For example if I want to run `kubectl get pods` then I can use the kgp alias:

```
alias kgp
```

So when I type this alias, and hit space.


```
kgp<space>
```

The globalias plugin get's triggered and it replaces the alias with the actual underlying command.

Pretty cool right. I'll be using these aliases a lot to cut down my typing and to get things done quicker.

Finally i have the minikube command. This is just another way to enable minikube autocompletion.

```
minikube tab-tab
```

Finally I've enabled the zsh-syntax-highlighting plugin. This is one of my favourite tools because it enables syntax highlighting on the command line. For example, an invalid command shows up in red:


```
ech
```

And turns green if it is valid, and it does other syntax highlighting such as highlighting things in double quotes:

```
echo "hello world"
```

This plugin does require a couple of extra steps to setup. So check the course notes for more info.