# Securing the cluster

All communication is done via the api.  The api uses https:


```bash
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.110:8443
KubeDNS is running at https://192.168.99.110:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

The api-server is the first point of contact when you interact with the kubecluster using kubectl. When the api component recieves an api request from kubectl, it needs to pass a 3-step-check before the request gets processed:

1. Authentication - i.e. provide valid credentials
2. Authorization - i.e. credentials have adequate permssions for the cluster and namespace in question. 
3. Admission Control - This is a long list of plugins, a pods plugin, a services plugin,....etc. Each plugin in turn assesses whether it can process the api request. E.g. the pods plugin will process any api requests that makes any pod changes. Note these Admission Control plugins are all ignored if the api request is a view request, e.g. kubectl get xxx. In which case kube-apiserver just queries etcd. 


## Authentication
Kubectl uses the credential info stored in the `~/.kube/config` to authenticate with the the api-server. This config file can store settings for multiple clusters, for example here's the connection details for connecting to a minikube cluster:

```bash
$ cat ~/.kube/config
...
clusters:
- cluster:
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
    server: https://192.168.99.110:8443
  name: minikube
...
contexts:
- context:
    cluster: minikube
    user: minikube
    namespace: default        # defaults to 'default' if this line is omitted
  name: minikube
...
users:
- name: minikube
  user:
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
...
current-context: minikube
```

The above entries was added by the minikube cli, behind the scenes when you ran the `minikube start` command. Note, you can refresh these entries with minikube:

```
$ minikube update-context
🙄  IP was already correctly configured for 192.168.99.110
```

## Certificate-Based Mutual Authentication


When connecting to a cluster (e.g. the minikube cluster), we need to specify 3 TLS files:

```bash
$ file /Users/schowdhury/.minikube/ca.crt
/Users/schowdhury/.minikube/ca.crt: PEM certificate
$ file /Users/schowdhury/.minikube/client.crt
/Users/schowdhury/.minikube/client.crt: PEM certificate
$ file /Users/schowdhury/.minikube/client.key
/Users/schowdhury/.minikube/client.key: PEM RSA private key
```


The following files are used to gain the api-server's trust of kubectl:

```
    client-certificate: /Users/schowdhury/.minikube/client.crt
    client-key: /Users/schowdhury/.minikube/client.key
```

Whereas the following file is used to establish the kubectl's trust, that kubectl has connected to the genuinely correct api-server:

```
    certificate-authority: /Users/schowdhury/.minikube/ca.crt
```

We want to avoid:


kubectl <---- man-in-the-middle-hacker ----> api-server

In this scenario let's assume the hacker has copies of the following files:

- kubectl's client.crt
- api-server's ca.crt

This is very likely because both these files are designed to be shared out. 


The key to establish there is no man in the middle. Here's the situation with the 2 entities:



kubectl <--------> api-server


They key to establshing trust between these two entities is the use of asymmetric encryption, in the form of tls certicates. 

Here's where each entity stands:

- kubectl - I will trust the api-server if it can prove that it has a copy of the ca.crt file's private key. 
- api-server - I will trust the kubectl if it can prove it has a copy of the client.crt file's private key. 


Here's how the handshake works:


1. kubectl creates an file that contains the following content:
   1. kubectl's client.crt
   2. randomly generated session id. 
2. kubectl encrypts this file using the api-server's ca.crt file. 
3. The kubectl then sends this encrypted file to the api-server. If the hacker intercepts this file, then it can't do anything with it since it's encrypted and it can't decrypt it. 
4. The api-server decrypts this file and finds the crt and session-id. the api-server validate the cert's signature to confirm it is a valid cert. However at this stage, the api-server doesnt trust kubectl yet, because the hacker might have created and sent the encrypted file. 
5. The api-server now create a file with the following content:
   1. initial-session-id
   2. randomly generated second session-id
6. The api-server encrypts this file with the kubectl's client.crt file. 
7. This encrypted file is then sent back to kubectl. If the hacker intercepts this file, then it still can't do anything with it because it can't decrypt it. Only the kubectl can decrypt it since it's the only entity to have the client.key file. 
8. The kubectl recieves and decrypts this file using it's client.key file. kubectl sees that this file has the original initial-session-id (which the hacker wasn't able to access). kubectl on seeing this initial-session-id is now satisfied and full trusts that it is communicating with it's target api-server. 
9. kubectl then encrypts it's api request, with the second-session-id and sends it to the api-server, here we're starting to use symmetric encryption. If the hacker intercepts this, then it still can't decrypt it since it doesn't know what the second-session-id is. At this point only kubectl and api-server knows what this second-session-id is. 
10. The api-server recieves the encrypted api-requests and successfully decrypts it with the second-session-id. This is enough for the api-server to fully trust the message-sender, i.e. the kubectl. The secure hand-shake is now complete, that means that authentication is also successful. The second-session-id will now be used to send encrypted communications between kubectl & api-server. The second-session-id key has an expiration time, when it expires, the whole handshaking process starts again in order to renew the second-session-id


