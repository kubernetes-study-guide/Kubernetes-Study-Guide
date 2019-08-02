# Helm


Charts is to helm, in a similar way to what rpms are to yum. Charts are usually distributed as targz files where, but once extracted they have a certain [chart folder structure](https://helm.sh/docs/developing_charts/#the-chart-file-structure):

```
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```








Helm charts comes in the form of tar files, which you can just download this tar file using [helm fetch](https://helm.sh/docs/helm/#helm-dependency):

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
│       │   ├── NOTES.txt      <= Use this to display useful info, e.g. url and port number to use along with default login credentials. # https://helm.sh/docs/chart_template_guide/#creating-a-notes-txt-file
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


Helm charts can have [dependencies](https://helm.sh/docs/helm/#helm-dependency) on other charts. E.g. the wordpress chart depends the the mariadb chart. This dependencies are specified in the requirements.yaml file:

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

While in the top level folder, run:

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

If the helm charts are local, then you can install it like this:

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
helm repo add ...
helm repo remove ...
helm repo list ...
helm repo update   # useful for cleangin up the cache
```

e.g.: 

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

$ helm repo list
NAME   	URL
stable 	https://kubernetes-charts.storage.googleapis.com
local  	http://127.0.0.1:8879/charts
bitnami	https://charts.bitnami.com/bitnami
```

# Create helm boilerplate

This is done using the [helm create](https://helm.sh/docs/helm/#helm-create) command: 

```bash
helm create --help  # has useful info
helm create hello-world
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

One file that doesn't get generated is the requirements.yaml. this stores chart dependenices. These dependencies are downloaded and installed in the charts folders. this file is also where the `helm dependency list` command gets it's info from. 


values.yaml has a bunch of default values. 

You can override these by just updating this values.yaml with new defaults. Not that good approach though.  A better way to override these values, is on the command like using the --set flag. 

```bash
helm install stable/wordpress --set key=value --set key=value ...
```

this commmand can end up getting quite long. An even better way is to create your own customs-values.yaml file which only includes the parts of the original yaml file you want to override, with the overrided value, and then use the --values flag to feed in your custom-values.yaml file.

```bash
helm install stable/wordpress --values=/path/to/customs-values.yaml
```




## References
[helm](https://helm.sh/)
[https://linuxhint.com/getting-started-kubernetes-helm-charts/](https://linuxhint.com/getting-started-kubernetes-helm-charts/)