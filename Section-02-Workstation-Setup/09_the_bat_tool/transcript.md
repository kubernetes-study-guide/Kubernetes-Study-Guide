The bat tool is an alternative to the cat command,

```
bat --help
```

but has some really cool features, the key one being that it supports syntax highlighting, for example here's what the output looks like when I bat out a yaml file:

```
ls -l
bat po...
```



How cool is that! When I first tried that it totally blew me away. The bat tool is something I never realised how badly I wanted it, until I saw it in action.

Not only that but it even displays the output in a nicely formatted way, with headers and line numbers.

And it even gets better, Theres a number of syntax highlighting themes to choose from and you can preview them using list-themes flag.

```
bat --list-themes
```

Once you've decided on a theme you can set it as your defualt in your zsh profile:

```
echo "BAT_THEME=DarkNeon"  >> ~/.zshrc
```

You should check out the documentation to learn about how to customise and configure this tool.


But I definitely recommended giving bat a try. for me it feels like I've been watching tv in black-and-white for several years and then discovering that there's a button on my remote control that turns on the color!!! Ok that might be a bit of an over exaggeration, but you get the picture. no-pun intended.