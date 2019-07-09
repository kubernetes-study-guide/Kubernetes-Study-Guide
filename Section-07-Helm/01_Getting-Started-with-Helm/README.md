# Helm


Install helm:

```bash
brew install kubernetes-helm
```

Helm is installed here:

```bash
$ helm home
/Users/schowdhury/.helm
```

In this folder, there is a repositories.yaml file. This is the equivalent to yum's .repo file. 


Next, confirm that you're on the right context (in order to install server side of helm, called Tiller, into the cluster):

```bash
$ kubectl config get-contexts
```

The next command will then setup tiller on this context:


```bash
helm init --history-max 200
Creating /Users/schowdhury/.helm
...
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /Users/schowdhury/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
```

This ends up installing the tiller pod:

```bash
$ kubectl get pods --namespace=kube-system | grep tiller
tiller-deploy-69d5cd79bb-qbtx7              1/1     Running   0          2m20s
```

Which is create by the following deployment:

```bash
$ kubectl get deployments --namespace=kube-system | grep tiller
tiller-deploy              1/1     1            1           9d
```

You can output the all the yaml definitions that were used to create the tiller app by running:

```bash
helm init --dry-run --debug
```

Tiller is the server side component of helm. 

Note, tiller will actually be deprecated in the next version of helm. Instead, you will just have helm interacting with the kube-apiserver directly without any tiller middleman. 

Now list your helm charts:

```bash
helm ls
```

Here's how to search for a helm chart in the public repos:

```bash
$ helm search --versions wordpress
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/wordpress	5.12.3       	5.2.1      	Web publishing platform for building blogs and websites.
...
```

The --versions flag list all available versions. 

You can list all available helm charts (aka helm packages) by running:

```bash
$ helm search
```

Now here's how to [install](https://helm.sh/docs/using_helm/#more-installation-methods) wordpress app:

```bash
helm install stable/wordpress --name codingbee-wp
```

This creates a number of kubernetes objects, pods, services, configmaps,...etc, and groups them into a 'release' and names that release using the --name flag. If you run this command again, then it will create a second release (but you have to pick a different name). You can check your release's status by running:



```bash
helm status release-name
```

The 'helm install' installed a chart which in turn created a number of kubernetes objects, i.e. deployments, services, ConfigMaps,...etc. You can use `kubectl get ...` command to list them out in turn. But that would also involve identifying which object was created by the release. Better yet you can do:


```bash
helm get release-name
```

This is equivalent to running multiple `kubectl .... -o yaml` in one go for each object that exists in the release. 



A helm chart comes with a set of default values, you print these defaults:


```bash
helm inspect values stable/mariadb
...
db:
  ## MariaDB username and password
  ## ref: https://github.com/bitnami/bitnami-docker-mariadb#creating-a-database-user-on-first-run
  ##
  user:
  password:
  ## Password is ignored if existingSecret is specified.
  ## Database to create
  ## ref: https://github.com/bitnami/bitnami-docker-mariadb#creating-a-database-on-first-run
  ##
  name: my_database
...
```

You can [override these default values](https://helm.sh/docs/using_helm/#customizing-the-chart-before-installing) by running:

```bash
$ cat << EOF > config.yaml
db:
  user: Sher
  password: password123
  name: codingbee_db
EOF
$ helm install -f config.yaml stable/mariadb
```

-f is short for --values (which you can think of as --file instead). An alternative to --values is --set. 


To [delete](https://helm.sh/docs/helm/#helm-delete) a release, do:

```bash
$ helm delete release-name --purge
```

To see a list of deleted releases, do:

```bash
helm list --deleted
```

To search for and install a particular version, do:

```bash
helm search --versions stable/wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                             
stable/wordpress        5.13.0          5.2.2           Web publishing platform for building blogs and websites.
stable/wordpress        5.12.6          5.2.2           Web publishing platform for building blogs and websites.
stable/wordpress        5.12.5          5.2.2           Web publishing platform for building blogs and websites.
stable/wordpress        5.12.4          5.2.2           Web publishing platform for building blogs and websites.
...
helm install --name codingbee stable/wordpress --version 5.0.0
```

Here's how to upgrade to a particular version:

```bash
$ helm upgrade lazy-ragdoll stable/wordpress --version 5.0.1
```

If you want to upgrade to the latest version then you omit the version flag. this causes the revision to go up:

```bash
$ helm ls
NAME        	REVISION	UPDATED                 	STATUS  	CHART          	APP VERSION	NAMESPACE
lazy-ragdoll	2       	Tue Jun 11 19:25:45 2019	DEPLOYED	wordpress-5.0.1	5.0.1      	default
```


## References
[helm](https://helm.sh/)
[https://linuxhint.com/getting-started-kubernetes-helm-charts/](https://linuxhint.com/getting-started-kubernetes-helm-charts/)