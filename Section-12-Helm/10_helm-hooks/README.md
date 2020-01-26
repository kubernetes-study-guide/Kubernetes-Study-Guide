# Helm hooks

Helm provides a [hook](https://helm.sh/docs/developing_charts/#hooks) mechanism to allow chart developers to intervene at certain points in a release’s life cycle. For example, you can use hooks to:

- Load a ConfigMap or Secret during install before any other charts are loaded.
- Execute a Job to back up a database before installing a new chart, and then execute a second job after the upgrade in order to restore data.
- Run a Job before deleting a release to gracefully take a service out of rotation before removing it.
Hooks work like regular templates, but they have special annotations that cause Helm to utilize them differently. In this section, we cover the basic usage pattern for hooks.


You can view what hooks came with a release by running the [helm get hooks](https://helm.sh/docs/helm/#helm-get-hooks) command:


```bash
helm get hooks RELEASE-NAME
```




## Reference

https://helm.sh/docs/developing_charts/#hooks

