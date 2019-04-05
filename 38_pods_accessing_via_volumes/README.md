# accessing pod metadata using volumes

Env data only gives you access to a limited amount of data, in particular it doesn't give you access to pod **labels** and **annotations** data. If you want to give your pods access to more data, then you do that using the `pod.spec.volumes.downwardAPI` volume.








## References

[https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)