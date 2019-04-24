# Service: External Name


you can setup a C-Name DNS entry (DNS Alias to another DNS Alias) by creating service objects of the type 'ExternalName':

```yaml

---
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: codingbee.net
  ports:
  - port: 443
    targetPort: 443
```


