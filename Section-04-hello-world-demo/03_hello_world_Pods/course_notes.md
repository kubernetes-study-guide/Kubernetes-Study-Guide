
However there are a few other things I wanted to show you in this vidoe before we start creating service objects. First, you can run commands inside your pod using the exec command:

```bash
kubectl exec pod-httpd -c cntr-httpd -- ls -l
```

Here, the -c flag says which container inside the pod we want to connect to. And everything after the double dash, tells kubectl what command we want to run inside the container. If we omit the double dash then kubectl could treat the -l flag as a kubectl flag rather than the a flag for the ls command .

Another important feature of the exec command is that it let's you start an interactive bash session inside your pod:


```bash
$ kubectl exec -it pod-httpd -c cntr-httpd -- /bin/bash
root@pod-httpd:/usr/local/apache2#
```

This command is similar to the equivalent docker exec command. The -it flag says that we want to create an interactive terminal. Once your inside, you can run another curl test, but we first need to install curl:


```bash
apt-get -y update
apt-get install -y curl
```

Now let's run the curl test:

```bash
root@pod-httpd:/usr/local/apache2# curl http://localhost
<html><body><h1>It works!</h1></body></html>
root@pod-httpd:/usr/local/apache2# curl http://127.0.0.1
<html><body><h1>It works!</h1></body></html>
root@pod-httpd:/usr/local/apache2# hostname --ip-address
172.17.0.8
root@pod-httpd:/usr/local/apache2# curl http://172.17.0.8
<html><body><h1>It works!</h1></body></html>
```

Ok so everything looks good here, let me exit it out now.

```bash
exit
```


Finally I wanted to show you some commands you can use to get more detailed info about your pods. The first one is the 'get pods' command again, but this time we set the output flag to 'yaml':


```bash
kubectl get pods pod-httpd -o yaml | less
```

Also notice that I specified the pod's name in the command. That's to tell kubectl to only retrieve info for that one pod.


This yaml output is essentially the full form version of the yaml file that we used to build this pod. It shows a lot of the defaults that were used since we didn't explicitly specify all these setting in our yaml file.

Another command that gives a lot of info is the describe command:

```bash
kubectl describe pod pod-httpd | less
```

This has a lot of the same output as we saw with the get command. However it does have some other interesting info, such as an event log at the bottom, this can be useful for troubleshooting.

Once you have finished using the pod, you can then delete it by using the 'delete' command:

```bash
kubectl delete pod pod-httpd
```

This can take a few seconds to complete.


Ok that's it for this video see you in the next one.
