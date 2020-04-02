# Course notes - Customizing the zsh command prompt

## Customizing the agnoster command prompt structure

Removing workstation name and set custom username:

```
USER=pick-a-user-name
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```

Source: https://github.com/agnoster/agnoster-zsh-theme/issues/39#issuecomment-307338817




## General guide to customizing zsh $PROMPT variable

Useful links to learning about the zsh $PROMPT syntax:

- https://scriptingosx.com/2019/07/moving-to-zsh-06-customizing-the-zsh-prompt/
- https://www.sitepoint.com/zsh-tips-tricks/
- https://hackernoon.com/how-to-trick-out-terminal-287c0e93fce0
- https://til.hashrocket.com/posts/f3093399d0-test-out-your-zsh-prompt
- https://medium.com/@oldwestaction/beautifying-your-terminal-with-zsh-prezto-powerlevel9k-9e8de2023046
- https://code.joejag.com/2014/why-zsh.html


### Custom Prompt that spans across 2 lines

Here's a very simple way to split the command prompt into 2 lines:

```
PROMPT="$PROMPT
$ "
```

Note, $PROMPT is an alias of $PS1 

This variable contains a new-line character at the end of the first line. This is what causes the PROMPT to split into 2 lines. I've also included a dollar character followed by a blank space on the second line, to make it look more like a standard command prompt.


Here's another variaton of the above example but this time using an arrow:


```
PROMPT="╭$PROMPT
╰➤ "
```

This includes an arrow for cosmetic purposes. I personally think this looks prettier.



Here's another example but with a fancier arrow:

```
PROMPT="%(?.╭.%{$fg[red]%}╭%{$reset_color%})$PROMPT
%(?.╰➤.%{$fg[red]%}╰➤%{$reset_color%}) "
```


This has enclosed the arrow inside a couple of if-else statements in the form of:

```
%(?.true-text.false-text)
```

Where '?' indicates the exit code.

For more info about this syntax, see the official docs:

[http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Conditional-Substrings-in-Prompts](http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Conditional-Substrings-in-Prompts)


The `%(x.true-text.false-text)` indicates an if-else statement. This decides on which expression to print based on the exit code (?).
