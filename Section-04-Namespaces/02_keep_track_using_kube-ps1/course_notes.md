With hostname and username:
```
PROMPT='╭─$(kube_ps1)%{$terminfo[bold]$fg[green]%}%n@%m %{$reset_color%}%{$terminfo[bold]$fg[blue]%}%~ %{$reset_color%}$(ruby_prompt_info)$(git_prompt_info)$(virtualenv_prompt_info)
╰─%B$%b '
```

Without username and hostname:

```
PROMPT='╭─$(kube_ps1)%{$reset_color%}%{$terminfo[bold]$fg[blue]%}%~ %{$reset_color%}$(ruby_prompt_info)$(git_prompt_info)$(virtualenv_prompt_info)
╰─%B$%b '
```

with green ticks and red crosses:

```
PROMPT='╭─%(?.%{$fg[green]%}%B √ %{$reset_color%}.%{$fg[red]%}%B X %{$reset_color%})$(kube_ps1)%{$reset_color%}%{$terminfo[bold]$fg[blue]%} %~ %{$reset_color%}$(ruby_prompt_info)$(git_prompt_info)$(virtualenv_prompt_info)
╰─%B$%b '
```