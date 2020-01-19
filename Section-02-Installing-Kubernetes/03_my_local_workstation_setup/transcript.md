Hello Everyone and welcome back.

In this course I'll be doing a lot of demos on my local workstation. And when you're following along you might find that things might look a little different on your end when compared to what you see on my screen. So I thought I'll briefly walk through my local    setup to help explain those differences.

To start with my workstation is an Apple macbook Pro. And i have macos catalina running on it.

```
gui demo
```

For my terminal, I'll be using iterm2. iterm2 has a loads of extra features when compared to the MacOS's default terminal. For example     I can split a iterm2 terminal into multiple smaller terminals.

```
click on iterm2 terminal
```


As for the shell itself, I'll be using zsh instead of bash.

```
echo $SHELL
```


zsh is basically the same as bash, but with a lot more feature. In 2019 zsh replaced bash to become the default shell in  MacOS. So if you're tempted in switching to zsh, then now would be a great time to jump ship. However all the commands I'll demo will work fine on both shells.

In my case not only have switched to using zsh, I've also turbocharged it installing the oh-my-zsh framework. This framework gives me access to a lot of extra features  including a bunch of handy aliases:

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


As for my code editor, I'll be using VS Code. VS Code has it's own integrated terminal. You might have noticed that my VS Code interface looks a little different to what you might be expecting, and that's because I'm using the Cobalt2 vscode theme. Check out this video's description for a link to that theme  .

That's it for this video, see you in the next one.

