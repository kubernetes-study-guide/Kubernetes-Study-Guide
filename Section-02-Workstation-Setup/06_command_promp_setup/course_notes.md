Removing workstation name and set you're name:

```
USER=pick-a-user-name
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```

Source: https://github.com/agnoster/agnoster-zsh-theme/issues/39#issuecomment-307338817




```
# https://github.com/jonmosco/kube-ps1
PROMPT='$(kube_ps1)'$PROMPT
PROMPT="╭─$PROMPT
╰─%(?.➤.%{$fg[red]%}➤%{$reset_color%}) "
```