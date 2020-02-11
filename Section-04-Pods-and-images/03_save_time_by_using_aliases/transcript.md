Hello everone and welcome back.

In this video I'm going to show you technique I use to do kubernetes work more quickly and easily. And that's by using Linux aliases. In case you don't know what linux aliases are, it's basically something that let's you create your own personal custom commands.

Ok, so earlier I demoed the kubectl run command. Now this is one of those commands I use all the time. However it's a really long command and repeatedly typing this command can get annoying and tedious. So to get round this problem I'm going to make use of aliases.


By the way I haven't specified anything after the double dash. I'll explain why I've done that, a bit later.

Now to create an alias for this command I'm going to first surround my command with double quotes. Then I'm going to call the alias command at the begining. Ok now I need to come up with a good alias, I know how about tp, as in testpod.

Hold on a second before I create this alias, let's first quickly check that there isn't a command already called tp, otherwise I'll end up with conflict.

Ok there is no command called tp, so it looks like we can go head with using  tp as an alias.

Ok our new alias is now ready for use. But lets first double check that tp is now definitely an alias. Ok that looks good.

This means that whenever I want to run this command, all I now have to do is run tp on the command line.

So let's give that a try. Now watch what happens when I hit space.

Wow did you see that? As soon as I hit space, the tp alias get's autoexpanded to the actual command that this alias was set to. Pretty cool right. This autoexpansion is actually a feature that's provide by the oh-my-zsh framework along with the globalias plugin. (pop: see video description for more info).


Ok now all I have to do is to set what command I want to run inside my testpod. For example to open a terminal inside my pod I'll just need to run bash. Ok that worked as planned so let's exit out of that.

Also if I want to run any other command, then I can take the same approach.

That's why I left the command blank after the double quote, so that I can choose what to run.

Another thing you can do using this alias is that you can quickly customise it when needed. For example let's say I want to create my testpod using the busybox image. Then I can just make the change, after the alias has been autoexpanded.

ok That's odd, that didn't work. It looks like the busybox image doesn't come with bash. In that case lets just try the basic bourne shell. ok that worked. Let's exit it out that now.

Now one important that might catch you out is that aliases only exists for the current terminal session. That means that this alias won't work in a new terminal session.

Luckily There's a simple fix for that, all you have to do is append your aliases to the end of your shell's config file. So everytime you start a new shell, these aliases get executed when you're new shell session is starting up.

After that your aliases will always be available. Let's open up a new terminal just to double check. Cool that worked. Ok let's close that again.

This is one of a handful of aliases I'll be using in this course, and I've added all my aliases in this video's description, so you can make use them as well if you want to.


By the way if you're a bash user then you need to append your alias commands to your .bashrc file rather than the .zshrc file.

Ok let's take a break here and I'll see you in the next video.








Other aliases:

```
echo 'alias wgp="watch -n1 kubectl get pods"' >> ~/.zshrc
```