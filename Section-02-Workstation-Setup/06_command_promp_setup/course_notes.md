```
# https://github.com/jonmosco/kube-ps1
PROMPT='$(kube_ps1)'$PROMPT
PROMPT="╭─$PROMPT
╰─%(?.➤.%{$fg[red]%}➤%{$reset_color%}) "
```