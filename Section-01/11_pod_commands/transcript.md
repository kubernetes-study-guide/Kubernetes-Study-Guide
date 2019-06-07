# Transcript

TODO: ....

Structure:
vscode
-> vscode
-> github: 

## vscode

The whole point of having containers is to run a Linux process inside them. That process in turn provides a service that is of value to us. You can find what that process is by taking a look inside a pod. For example let's create our hello-world pod again

```bash
tree configs/
code 
```




But what start's that process in the first place? The answer is, commands.




The main reason we use containers is so to run a process that provides a service that's of value to us. This process can be in the form of a command or a shell script. 

if a image doesn't come with a predefined command baked into it, then you need to define an ongoing command in the pod definition instead. 