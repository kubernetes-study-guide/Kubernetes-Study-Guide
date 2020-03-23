Alright. Let's take a look at my terminal setup. 

```
click rocket icon - then search for 'term'
```


In this course I won't be using my macos's builtin terminal. Instead I've installed a popular alternaive called iterm2. It shows iterm on the laaunch screen here, but it actually called iterm2 after I've opened it. 

```
open terminal -> hover over the header
```


iterm2 has a lot of extra capabilites and customization options, but the feature I'll be using most often is the ability to easily split my terminal into smaller terminals.

```
do vertical and horizontal splits
```

So if your also a mac user then you might like to give iterm2 a try. 


For my shell I'll be using zsh.

```
echo $SHELL
```

zsh is like bash but with a lot more features. In the year 2019 zsh replaced bash to become the default shell in macos. So if you're thinking of making the switch to zsh then now would be a great time to jump ship. In my experience nearly everything I used to do on bash, works on zsh as well. However if there are any differences then I'll mention them in the course notes.  

I've also enhanced zsh further my installing the oh-my-zsh framework. 

It's dead easy to insteall, all you have to do is run a curl command. 


This framework gives you access to a lot of cools themes and plugins. 







and you can set these up by going into your .zshrc file,.... In my case I'm using the agnoster theme. 





And I've activated several plugins entering them in the plugins section. 






This framework gives me access to a lot of extra features including a bunch of handy aliases:

```
alias
```

For example if I want to run "kubectl get pods", then I can just use the kgp alias followed by space:

```
kgp<space>
```

Hitting space ends up replacing the alias with the actual underlying command it referenced to.

Pretty cool right. I'll be using these aliases a lot to cut down my typing and to get things done quicker.

By the way, these aliases are only available after activatin some om-my-zsh plugins. To do that I had to go to my home directory and open up the zsh config file, which is called .zshrc. In this file there's a section where you can select your plugins.

So if you're already a zsh user but haven't used this framework before, then I definitely recommend giving it a try. It's dead easy to install and you can find the install instructions in this video's description.
