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



Modify PROMPT to span 2 lines, and add a dynamic arrow head to indicate the exit code of the previous command.


## General guide to customizing ZSH $PROMPT variable

Useful links to learning about the zsh $PROMPT:

- https://scriptingosx.com/2019/07/moving-to-zsh-06-customizing-the-zsh-prompt/
-

```
PROMPT="╭$PROMPT
╰%(?.➤.%{$fg[red]%}➤%{$reset_color%}) "
```

This variable contains a new-line character at the end of the first line. This is what causes the PROMPT to split into 2 lines.

The second line has syntax in the form of:

```
%(?.<success expression>.<failure expression>)
```

The `%(...)` indicates an if-else statement. This decides on which expression to print based on the exit code (?).








```
# https://github.com/jonmosco/kube-ps1
PROMPT='$(kube_ps1)'$PROMPT
PROMPT="╭─$PROMPT
╰─%(?.➤.%{$fg[red]%}➤%{$reset_color%}) "
```