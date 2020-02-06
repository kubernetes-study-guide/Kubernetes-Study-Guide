
Hello everone and welcome back.

Ok, so earlier I demoed the kubectl run command. Now I use this command all the time in my day to day work. However it's a really long command and typing this command out over and over again can get annoying and tedious. Luckily help is at hand in the form of linuxes aliases.


In Linux, aliases are used for assigning aliases to your commands. So when you want run a particular command then you can just type out the alias instead.

So let's try out aliases by using this kubectl run command as our example. Notice that I haven't put anything after the double dash. That's so that I can choose what to write here later on.

But for now let's create our alias. To do that I'm going to first surround my command with double quotes. Then I'm going to use the alias command to create an alias called tp and set it to this kubectl run command. in my case tp is just short for testpod.

Hold on a second before I create this alias, let me first quickly check that there isn't a command already called tp, otherwise it'll cause conflicts.

Cool, so it looks like we can use tp.

Ok our new alias tp is now ready for use. But lets first double check that tp is now an alias. Ok that looks good. Let's now give it try.

Now to do that, I'm going to type tp, then hit space. As you can see, as soon as I hit space, the tp alias get's autoexpanded to the actual command the tp alias was set to. This autoexpansion normally doesn't happen with a standard zsh or bash sessions.




Other aliases:

```
echo 'alias wgp="watch -n1 kubectl get pods"' >> ~/.zshrc
```