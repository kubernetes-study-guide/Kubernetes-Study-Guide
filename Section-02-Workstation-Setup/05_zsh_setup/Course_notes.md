# Course Notes: Zsh setup

- [oh-my-zsh framework](https://github.com/ohmyzsh/ohmyzsh)
- [oh-my-zsh themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)
- [oh-my-zsh plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)

To change default shell on mac, run:

```
chsh -s /bin/zsh
```



zsh has now become the default shell on MacOS - [https://www.theverge.com/2019/6/4/18651872/apple-macos-catalina-zsh-bash-shell-replacement-features](https://www.theverge.com/2019/6/4/18651872/apple-macos-catalina-zsh-bash-shell-replacement-features)
[https://support.apple.com/en-gb/HT208050](https://support.apple.com/en-gb/HT208050)



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


prompt setup:

https://scriptingosx.com/2019/07/moving-to-zsh-06-customizing-the-zsh-prompt/
https://www.sitepoint.com/zsh-tips-tricks/
https://hackernoon.com/how-to-trick-out-terminal-287c0e93fce0
https://til.hashrocket.com/posts/f3093399d0-test-out-your-zsh-prompt
https://medium.com/@oldwestaction/beautifying-your-terminal-with-zsh-prezto-powerlevel9k-9e8de2023046
https://code.joejag.com/2014/why-zsh.html