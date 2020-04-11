https://github.com/jonmosco/kube-ps1



https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kube-ps1


actually keep this prompt throughout the course. Becuase you need it to show whether you're using digital ocean or minikube cluster. 


# should talk about this later. 
https://github.com/ahmetb/kubectx


mac users have to also add in the following to the kubernetes image icon working:

```
KUBE_PS1_SYMBOL_ENABLE=true
KUBE_PS1_SYMBOL_DEFAULT=$'\u2638\ufe0f '
KUBE_PS1_SYMBOL_USE_IMG=false
```

Source: https://github.com/jonmosco/kube-ps1/issues/117#issuecomment-603557023
