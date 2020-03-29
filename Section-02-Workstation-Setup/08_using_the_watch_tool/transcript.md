The watch command let's you run the same command over and over and displays it's output. You run it by typing watch followed by the command your interested in.

```
watch ls -l
```

The watch utility runs your command every 2 seconds by default. At the moment it's showing that this folder is empty. So lets see what happens when I create a new testfile.

As you can see it now shows up in the output.

This let's you watch for any changes in the output in near realtime, that's where it get's it's name from.

However you can change this interval using the minus-n flag. For example here I'm setting it to every 3 seconds.

```
watch -n3 ls -l
```

I'll be using the watch command a lot to monitor for changes that happens on our kube cluster while performing kubernetes demos.