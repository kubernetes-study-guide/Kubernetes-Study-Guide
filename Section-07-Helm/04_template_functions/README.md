# Variables

If you want to modify your variables before they get inserted into the template, e.g. put your variables inside quotes, then you need to use [template functions](https://helm.sh/docs/chart_template_guide/#built-in-objects). 


```bash
$ helm install . --dry-run --debug 
...
spec:
  containers:
  - name: "cntr-nginx"
    image: nginx:stable
...
```

There are a lot of functions available:

- [https://godoc.org/text/template](https://godoc.org/text/template)
- [http://masterminds.github.io/sprig/](http://masterminds.github.io/sprig/)