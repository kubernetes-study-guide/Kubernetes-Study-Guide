# transcript

Hello everyone

So far we created objects by feeding yaml files into kubectl.

Yaml is just a markup language like xml or json. The Yaml syntax is used for writing data in a structured way. I recommend taking look at the Ansible website's yaml guide if you want to learn about the syntax.

Basically YAML is based on a key-value system. Where the key is a string, and the value is essentially a placeholder that can hold all sorts of things, such as:

- A string
- An list of values
- Another key value pair
- or even A dictionary (a set of key value pairs).

One of the nice things about yaml is that it's relatively human readable,especially when compared to xml or json. 

The yaml files we created so far had following general structure:

```yaml
apiVersion: xxx
kind: xxxxx
metadata:
  {blah blah blah}
spec:
  {blah blah blah}
```

By the way, key names, such as 'apiVersion', are case sensitive. They are also space sensitive. So need to always make sure all your indents are correct.


The apiVersion, kind, metadata, and spec, are required fields for all kubernetes object types.

So as we saw earlier,

the 'kind' setting is used to set the object type, such as pods, services,..etc.


The 'apiVersion' sets which part of the kubernetes api to access. This depends on the object kind. For example, if the kind is 'Pod' then this field needs to be set to 'v1', as specified in the explain command.

```bash
$ kubectl explain pod
KIND:     Pod
VERSION:  v1
...
```


the metadata' section is used to store info to help uniquely identify the object, such as the object's name.

The 'spec' section is where you set object's detailed specifications.
