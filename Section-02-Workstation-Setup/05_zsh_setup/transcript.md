So I'm using iterm2 to access my shell. As for the shell itself, I'm using zsh.

```
echo $SHELL
```

zsh is like bash but with lots of extra features. In the year 2019 Apple made zsh the default shell in macOS, before it used to be bash. So if you're thinking of making the switch to zsh then now would be a great time to jump ship, if you haven't already done so. In my experience nearly everything I used to do on bash, works just as well in zsh, if not better.

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


In my case I've activated the gianu theme. However some themes do require some extra work to setup, such as installing font packages.



I've also activated several plugins by adding their name to the plugins section.

Let's go over these plugins.

The docker plugin simply enables the docker autocomplete feature.

````
vertical split, vim in right terminal
````

Similarly the minikube plugin enable autocomplete for minikube.

```
minikube tab-tab
```

You might recall I've already setup autocompletion back when I first installed minikube. So this is just an alternative way of doing the same thing.


The kubectl plugin enables autocompletion for kubectl.


```
kubectl tab-tab
```

Once again I've already setup kubectl autocompletion in a previous video, so this is yet another way of enabling it.


However the kubectl plugin also sets up bunch of handy aliases.

```
aliases | grep kubectl
```


These aliases are a massive time saver but you might struggle to follow the demos if I'm using these cryptic looking aliases. That's why I've also installed the globalias plugin. This plugin auto expands an alias to the actual underlying command. Let me show you what I mean.



Let's say I want to run `kubectl get pods`, then I can use the kgp alias:

```
alias kgp
```

So to use this alias, I first type it on the command line,


```
kgp
```

Then to trigger th globalias plugin, all I have to do, is hit space.

```
<space>
```

globalias then automaically swaps out the alias with the actual underlying command. Pretty cool right.

Finally I've enabled the zsh-syntax-highlighting plugin. This enables syntax highlighting on the command line. For example, an invalid command shows up in red:


```
ech
```

And turns green if it is valid. It also does other syntax highlighting such as highlighting things in double quotes:

```
echo "hello world"
```

Most of these plugins do require some extra work to set them up. For example the docker plugin won't work if you haven't installed. So I recommend reading the plugin's install instruction when installing them.