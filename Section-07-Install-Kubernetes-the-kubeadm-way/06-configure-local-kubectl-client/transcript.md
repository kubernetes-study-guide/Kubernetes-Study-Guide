It's great that we can now use kubectl when ssh into a master kubemaster. But it's bad practice to ssh into you kubemasters just to use kubectl. Instesad you should use your workstation's kubectl client to interact with your cluster. To do that we need to configure our ~/.kube/config. We'll do that by running the following commands:


On master run:

```
cat ~/.kube/config
```

As you can see we have 3 long strings. they are base64 encoded. we need to provide all three pieces of info to our mac's kubectl so that it can authenticate with our new cluster. Why 3, why not 2 or just one? The answer to that is the kubectl uses asymmetric encryption along with mutual authentication. See course notes if you want to know more. 



on mac:

```
mkdir ~/.kube-digitlocean-certs
cd 
```

THen run:


```
echo "xxxx" | base64 --decode
```

Here we can see the decoded content. Let's pipe that to a file. 

```
echo "xxxx" | base64 --decode > certificate-authority-data.txt
```


Now repeat with the other 2:

```
echo "xxxx" | base64 --decode > client-certificate-data.txt

echo "xxxx" | base64 --decode > client-key-data.txt
```


Now let's create entry in our own ~/.kube/config file:

```
$ kubectl config set-cluster digital-ocean-cluster \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://188.166.136.150:6443
```

```
$ kubectl config set-credentials digital-ocean-admin \
    --client-certificate=client-certificate-data.txt \
    --client-key=client-key-data.txt
```

This command ties the above to together:

```
$ kubectl config set-context digital-ocean \
    --cluster=digital-ocean-cluster \
    --user=digital-ocean-admin
```

Now we activate our context:

```
kubectl config use-context digital-ocean --namespace default
```

Here we are saying that we want to use use-context, that in turn means that kubectl will interact with the xxx cluster using the xxx user credentials. We've also said that we want to use 'default' as our current namespace. Basically this command is saying which cluster to connect to with what credentials to make that connection with. 

