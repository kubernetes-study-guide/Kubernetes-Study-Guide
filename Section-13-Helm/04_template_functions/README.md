# Variables

If you want to modify your variables before they get inserted into the template, e.g. put your variables inside quotes, then you need to use [template functions](https://helm.sh/docs/chart_template_guide/#built-in-objects). 


```bash
$ helm install charts/01nginx --dry-run --debug 
...
spec:
  containers:
  - name: "cntr-nginx"
    image: nginx:stable
...
```

These functions comes from [3 places](https://helm.sh/docs/developing_charts/#templates-and-values):

- [https://godoc.org/text/template](https://godoc.org/text/template)
- [http://masterminds.github.io/sprig/](http://masterminds.github.io/sprig/)
- [https://helm.sh/docs/developing_charts/#chart-development-tips-and-tricks](https://helm.sh/docs/developing_charts/#chart-development-tips-and-tricks) - 'inlcude' and 'required' are special functions developed specifically for helm 