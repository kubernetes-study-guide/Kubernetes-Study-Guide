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


Next, confirm that you're on the right context:

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

Tiller is the server side component of helm. 


Now list your helm charts:

```bash
helm ls
```

Here's how to search for a helm chart in the public repos:

```bash
$ helm search wordpress
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/wordpress	5.12.3       	5.2.1      	Web publishing platform for building blogs and websites.
```

Now install this:

```bash
helm install stable/wordpress
```

This creates a number of kubernetes objects, pods, services, configmaps,...etc, and groups them into a 'release' and it names that release. If you run this command again, then it will create a second release. 

To delete a release, do:

```bash
$ helm delete release-name --purge
```


To install a particular version, do:


```bash
helm install stable/wordpress --version 5.0.0
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

Helm charts comes in the form of tar files, which you can just download:

```bash
$ helm fetch stable/wordpress
$ helm fetch --untar stable/wordpress
$ ll
total 64
drwxr-xr-x  10 schowdhury  staff    320 11 Jun 19:31 wordpress
-rw-r--r--   1 schowdhury  staff  29134 11 Jun 19:32 wordpress-5.12.3.tgz
```

The content of the wordpress folder is:

```bash
$ tree .
.
├── Chart.yaml
├── README.md
├── charts
│   └── mariadb
│       ├── Chart.yaml
│       ├── OWNERS
│       ├── README.md
│       ├── files
│       │   └── docker-entrypoint-initdb.d
│       │       └── README.md
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── _helpers.tpl
│       │   ├── initialization-configmap.yaml
│       │   ├── master-configmap.yaml
│       │   ├── master-pdb.yaml
│       │   ├── master-statefulset.yaml
│       │   ├── master-svc.yaml
│       │   ├── role.yaml
│       │   ├── rolebinding.yaml
│       │   ├── secrets.yaml
│       │   ├── serviceaccount.yaml
│       │   ├── slave-configmap.yaml
│       │   ├── slave-pdb.yaml
│       │   ├── slave-statefulset.yaml
│       │   ├── slave-svc.yaml
│       │   ├── test-runner.yaml
│       │   └── tests.yaml
│       ├── values-production.yaml
│       └── values.yaml
├── requirements.lock
├── requirements.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── externaldb-secrets.yaml
│   ├── ingress.yaml
│   ├── pvc.yaml
│   ├── secrets.yaml
│   ├── svc.yaml
│   ├── tests
│   │   └── test-mariadb-connection.yaml
│   └── tls-secrets.yaml
└── values.yaml

7 directories, 38 files
```


Helm charts can have dependencies on other charts. E.g. the wordpress chart depends the the mariadb chart. This dependencies are specified in the requirements.yaml file:

```bash
$ cat requirements.yaml
dependencies:
- name: mariadb
  version: 5.x.x
  repository: https://kubernetes-charts.storage.googleapis.com/
  condition: mariadb.enabled
  tags:
    - wordpress-database
```

these dependencies are stored in the charts folder:

```bash
$ tree charts/
charts/
└── mariadb
    ├── Chart.yaml
    ├── OWNERS
    ├── README.md
    ├── files
    │   └── docker-entrypoint-initdb.d
    │       └── README.md
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── initialization-configmap.yaml
    │   ├── master-configmap.yaml
    │   ├── master-pdb.yaml
    │   ├── master-statefulset.yaml
    │   ├── master-svc.yaml
    │   ├── role.yaml
    │   ├── rolebinding.yaml
    │   ├── secrets.yaml
    │   ├── serviceaccount.yaml
    │   ├── slave-configmap.yaml
    │   ├── slave-pdb.yaml
    │   ├── slave-statefulset.yaml
    │   ├── slave-svc.yaml
    │   ├── test-runner.yaml
    │   └── tests.yaml
    ├── values-production.yaml
    └── values.yaml

4 directories, 23 files
```

While in the same directory as the chart.yaml, run:

```bash
$ helm dependency list
NAME   	VERSION	REPOSITORY                                       	STATUS
mariadb	5.x.x  	https://kubernetes-charts.storage.googleapis.com/	unpacked
```

If you empty out the charts folder and run the above command again, you get:


```bash
$ helm dependency list
NAME   	VERSION	REPOSITORY                                       	STATUS
mariadb	5.x.x  	https://kubernetes-charts.storage.googleapis.com/	missing
```

To pull down the missing dependencies you run the following while in the helm's root folder:

```bash
$ helm dependency update
Hang tight while we grab the latest from your chart repositories...
...Unable to get an update from the "local" chart repository (http://127.0.0.1:8879/charts):
	Get http://127.0.0.1:8879/charts/index.yaml: dial tcp 127.0.0.1:8879: connect: connection refused
...Successfully got an update from the "stable" chart repository
Update Complete.
Saving 1 charts
Downloading mariadb from repo https://kubernetes-charts.storage.googleapis.com/
Deleting outdated charts
```

If the helm charts are locally, then you can install it like this:

```bash
$ helm install .
```

This is a good appraoch if you git clone the helm source code, and then run this command. In this scenario you skip using tar files altogether. However you can also feed in tar files too:

```bash
$ helm install /path/to/helmchart.tar.gz
```



## Helm Repos

Helm downloads tar files from 'helm repos'. Similar to you yum downloads from rpm repos. However instead to .repo files, helm uses repositories.yaml file:

```bash
$ cat ~/.helm/repository/repositories.yaml 
apiVersion: v1
generated: "2019-06-11T19:03:36.605906+01:00"
repositories:
- caFile: ""
  cache: /Users/schowdhury/.helm/repository/cache/stable-index.yaml
  certFile: ""
  keyFile: ""
  name: stable
  password: ""
  url: https://kubernetes-charts.storage.googleapis.com
  username: ""
- caFile: ""
  cache: /Users/schowdhury/.helm/repository/cache/local-index.yaml
  certFile: ""
  keyFile: ""
  name: local
  password: ""
  url: http://127.0.0.1:8879/charts
  username: ""
```

You can crud this file using helm by using commands like:

```bash
helm repo add
helm repo remove
helm list
helm repo update   # useful for cleangin up the cache
```


# Create helm boilerplate

This is done using 

```bash
$ helm create hello-world
```
This creates:

```bash
$ tree ./hello-world
./hello-world
├── Chart.yaml          # mainly contains meta data
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml     # this contains default values. These are used 
                    # to render the yaml files stored in the templates 
                    # folder. 

3 directories, 8 files
```

One file that doesn't get requreted is the requirements.yaml. this stores chart dependenices. These dependencies are downloaded and installed in the charts folders. this file is also where the `helm dependency list` command gets it's info from. 


values.yaml has a bunch of default values. you can override these values on the command like using the --set flag. 

```bash
helm install --set key=value --set key=value ...
```





## References
[helm](https://helm.sh/)
[https://linuxhint.com/getting-started-kubernetes-helm-charts/](https://linuxhint.com/getting-started-kubernetes-helm-charts/)