
For my shell I'll be using zsh.

```
echo $SHELL
```

zsh is like bash but with a lot more features. In the year 2019 Apple made zsh the default shell in macOS, before it used to be bash. So if you're thinking of making the switch to zsh then now would be a great time to jump ship. In my experience nearly everything I used to do on bash, works just as well in zsh, if not better.

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

You can set your themes and plugins by going into your .zshrc file.


```
swipe
vim ~/.zshrc
```


In my case I've activated the gianu theme.


I've also activated several plugins by entering their name in the plugins section.

Let's go over some of these plugins

The docker plugin simply enables the docker autocomplete feature.

````
vertical split, vim in right terminal
````

Similarly the minikube plugin enable autocomplete for minikube.

```
minikube tab-tab
```

You might recall I've already setup minikube autocompletion back when I first installed minikube. So this is just an alternative way of doing the same thing.


The kubectl plugin enables autocompletion for kubectl.


```
kubectl tab-tab
```

I've already setup kubectl autocompletion in a previous, so this is yet another way of enabling it.


However the kubectl plugin also sets up bunch of handy aliases.

```
aliases | grep kubectl
```


These aliases are a massive time saver but you might struggle to struggle to follow what I'm doing if I'm using these cryptic looking aliases. That's why I'm also going to use the globalias plugin. This auto expands an alias to the actual underlying command. Let me show you what I mean.



Let's say I want to run `kubectl get pods`, then I can use the kgp alias:

```
alias kgp
```

So to use this alias, I first type it out,


```
kgp
```

Then to trigger teh globalias plugin, all I now have to do is hit space. The globalias plugin then automaically swaps out the alias with the actual underlying command.



```
<space>
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