The watch tool let's you run the same command over and over and displays it's output.

```
watch --help
```


You run it by typing watch followed by the command your interested in.

```
watch ls -l
```

The watch utility runs your command every 2 seconds by default. At the moment it's showing that this folder is empty. So lets see what happens when I create a new testfile.

```
# in another terminal
touch testfile.txt
```

As you can see it now shows up in the output.

So in effect tool let's you watch for any changes in the output in near realtime, that's where it get's it's name from.

Also if you wanted to, you can thange this 2 second default interval using the minus-n flag. Let me first controlc to exit out of watch

```
control+c
```

Ok now here's how to use the -n flag:

```
watch -n3 ls -l
```

For example here I'm setting it to every 3 seconds.

Now my command is being 3 seconds.

```
cntrlc
up-arrow
```

By the way you can also put a space after the minus-n flag. it still works the same way.
