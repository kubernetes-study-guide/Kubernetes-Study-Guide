


By creating my own personal bin folder helps me to keep things organised and downloading executables into it, help me to keep things organised. For example when I run 


```
which kubectl
```




and see that it's in my personal bin folder, then it means that this tool is something i installed manually rather than it being installed by a package manager, such has homebrew, yum, dnf,,,.etc. 

Now you might be wondering why not just install it using a package manager such as homebrew:


```
brew install kubectl
```

That's by installing it manually you can have multiple versions of kubectl client. For example:

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/darwin/amd64/kubectl ... 
```

chmod +x ...


kubectl-v1.17.0 version



