Hello Everyone and welcome back.

In this course I'll be doing a lot of demos on my local workstation. And when you're following along you might find that your screen might look a little different when compared to what you see on my screen. So in this video I'm going to briefly walk through my local workstations setup to help explain those differences.

To start with my workstation is an Apple macbook Pro. And i have macos catalina running on it.

```
gui demo
```

For my terminal application I'll be using iterm2. That's because iterm2 has a loads of extra features when compared to the default terminal that comes shipped with Apple MacOS, one example is that I can split a iterm2 terminal into multiple smaller terminals.

```
click on iterm2 terminal
```


As for the shell itself, I'll be using zsh instead of bash.

```
echo $SHELL
```


That's because zsh is more feature-rich when compared to bash and it has become the default shell in MacOS since 2019. However all the commands I'll demo will work fine on both shells.

I've also turbocharged my zsh by installing the oh-my-zsh framework. This framework gives me access to a lot of extra features including a bunch of handy aliases:

```
alias
```

For example if I want to run "kubectl get pods", then I can just use the kgp alias followed by space:

```
kgp<space>
```

Hitting space ends up replacing the alias with the actual underlying command it referenced to.

Pretty cool right. I'll be using these aliases quite a lot to cut down my typing and to get things done quicker.

By the way, these aliases are only available after the relevant om-my-zsh plugins are activated. To do that I had to go to my home directory and open up the zsh config file, which is called .zshrc. In this file there's a plugin section where I added in my plugins.

So you haven't already started using the zsh along with the oh-my-zsh framework, then I definitely recommend giving it a try. It's dead easy to install and you can find the install instructions in this video's description.


As for my code editor, I'll be using VS Code. VS Code has it's own integrated terminal which I'll make use of as well. You might have noticed that my VS Code interface looks a little different to yours and that's because I'm using the Cobalt2 vscode theme. Check out this video's description for more information about that too.