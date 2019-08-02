# Variables

I created this demo by starting with the boilerplate:

```bash
helm create nginx
```


and then stripping most of it out. Then I ran:


```bash
helm lint .
```

To make sure I haven't broken anything. I only have one templatised yaml file in this chart:


```bash
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Release.Name }}
  labels:
    Description: {{ .Chart.Description }}
spec:
  containers:
  - name: {{ .Values.container_name }}
    image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
    ports:
    - containerPort: 80
```

[https://helm.sh/docs/chart_template_guide/#adding-a-simple-template-call](https://helm.sh/docs/chart_template_guide/#adding-a-simple-template-call)


This template pulls in variables values from 3 places:

- values.yaml
- charts.yaml
- the release [builtin-object](https://helm.sh/docs/chart_template_guide/#built-in-objects). These are metadata that are generated during the helm-install command. 


Now let's perform a dry run, to see what our rendered yaml file looks like:

```bash
$ helm install . --dry-run --debug
```

Note: The built-in values always begin with a capital letter. This is in keeping with Go’s naming convention. When you create your own names, you are free to use a convention that suits your team. Some teams, like the Helm Charts team, choose to use only initial lower case letters in order to distinguish local names from those built-in. In this guide, we follow that convention.

So let's override the randomly generated name:

```bash
$ helm install . --name=codingbee
NAME:   codingbee
LAST DEPLOYED: Wed Jul 10 16:55:50 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod
NAME       READY  STATUS             RESTARTS  AGE
codingbee  0/1    ContainerCreating  0         0s
```

Then check:

```bash
$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
codingbee   1/1     Running   0          73s
```

Now we can do a few checks:

```bash
helm get manifest codingbee
helm get values codingbee --all
```

Now let's delete

```bash
$ helm delete codingbee
release "codingbee" deleted
$ helm delete codingbee --purge
release "codingbee" deleted
```



You can [override the content in the and values.yaml](https://helm.sh/docs/chart_template_guide/#values-files) on the command line. 


```bash
helm install . --dry-run --debug --set image.repository=httpd
```

You can use the --set flag for multiple overrides:


```bash
helm install . --dry-run --debug --set image.repository=httpd --set container_name=cntr-httpd
```

or feed these in a custom yaml file, here's a quick shorthand:

```bash
cat <<EOF | helm install . --dry-run --debug -f -
container_name: cntr-httpd

image:
  repository: httpd
EOF
```

[Capabilities and Template](https://helm.sh/docs/chart_template_guide/#built-in-objects) are a couple of other builtin objects:

```bash
$ cd charts/02nginx/
$ helm install . --dry-run --debug | less


$ helm install . --name=codingbeewp --dry-run --debug | grep KubeVersion
    capabilityKubeVersion: v1.15.0

$ helm install . --name=codingbeewp --dry-run --debug | grep helmTemplateName
    helmTemplateName: 02nginx/templates/pod.yaml


```












