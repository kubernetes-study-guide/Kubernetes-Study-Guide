do in iterm2, have three panels open

2 columns, second column has two panels, up and down. 

watch kubectl get replicasets
watch kubectl get pods

actually use this to watch everything instead: 



Finally....By the way, you can also use plugins:

install:
https://github.com/ahmetb/kubectl-tree   ("watch" command doesn't work properly with this)

You can follow the documentation if you want to install this plugin. It's quite easy to install. 



on the surface deployments and replicasets look like they do the same thing.

When I first started about deployments I thought deployments was pointless, I mean why bother with using it if you can just create a replicaset directly. It just seemd liked deployments just adds another layer of complexity. 




