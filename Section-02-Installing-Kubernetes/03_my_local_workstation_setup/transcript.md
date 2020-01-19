In this course I'll be doing a lot of demos on my local workstation. So when you're following along you might find that things look a little different when compared to what you see on my screen. So in this video I'm going to briefly walk through my local workstation's setup so that you find it easier to follow the demos.

To start with my workstation is a Apple macbook Pro 2015 edition.

For my terminal application I'll be using iterm2. That's because iterm2 has a loads of extra features when compared to the default terminal that comes shipped with Apple MacOS, one example is that I can split a iterm2 terminal into multiple smaller terminals.

```
click on iterm2 terminal
```


As for the shell itself, I'll be using zsh instead of bash.

```
echo $SHELL
```


That's because zsh is more feature-rich when compared to bash and it has become the default shell in MacOS since 2019. However all the commands I'll demo will work on both shells.

I've also turbocharged zsh by installing the oh-my-zsh framework. This framework gives me access to a lot of extra features including a bunch of handy aliases:

```
alias
```

For example if I want to run "kubectl get pods", then I can just use the kgp alias followed by space:

```
kgp<space>
```

Pretty cool right. I'll be using these aliases quite a lot to save time and get things done quicker.

So you haven't already started using the zsh and the oh-my-zsh then I definitely recommend giving it a try. The install instructions are easy to following and in my case I installed it by running a single curl command.

By the way, these aliases are only available after the relevant om-my-zsh plugins are activated. To do that I had to go to my home directory and open up the zsh config file, which is called .zshrc. In this file there's a plugin section where I added in my plugins.




As for my code editor, I'll be using VS Code. VS Code has it's own integrated terminal which I'll make use of as well. You might have noticed that my VS Code interface looks a little different to yours and that's because I'm using the Cobalt2 vscode theme.






