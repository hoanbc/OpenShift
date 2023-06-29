#### Install Quay Opeator
```sh
Console -> Operators -> OperatorHub -> Red Hat Quay -> Install (Use testvn-quay namespace)
```
#### Create secret for object storage use minio
```sh
#config.yaml
DISTRIBUTED_STORAGE_CONFIG:
    minio:
        - RadosGWStorage
        - access_key: t8TJPsGMhvfLl7CYBBIz
          bucket_name: quay
          hostname: s3.testvn.click
          is_secure: true
          port: "443"
          secret_key: G4IYxJJj0lrX5xiLmhR19aDJvfSA6aXTgIV1NyBU
          storage_path: /datastorage/registry
DISTRIBUTED_STORAGE_DEFAULT_LOCATIONS:
    - minio
DISTRIBUTED_STORAGE_PREFERENCE:
    - minio
SERVER_HOSTNAME: quay.testvn.click
```
#### Create secret for object storage use S3
```sh
#config.yaml
SERVER_HOSTNAME: quay.testvn.click
DISTRIBUTED_STORAGE_CONFIG:
    aws:
        - S3Storage
        - host: s3.ap-southeast-1.amazonaws.com
          port: "443"
          s3_access_key: AKIAWU2PE4D11144FG
          s3_bucket: quay-testvn
          s3_secret_key: NPVSx1JFtb11jUoRAqEbiyM6
          storage_path: /datastorage/registry
DISTRIBUTED_STORAGE_DEFAULT_LOCATIONS:
    - aws
DISTRIBUTED_STORAGE_PREFERENCE:
    - aws
```
#### Create Quay Registry
```yaml
#Console -> Operators -> OperatorHub -> Installed Operators-> Red Hat Quay -> Quay Registry -> Create
apiVersion: quay.redhat.com/v1
kind: QuayRegistry
metadata:
  name: registry
  namespace: testvn-quay
spec:
  components:
    - kind: clair
      managed: false
    - kind: postgres
      managed: true
    - kind: objectstorage
      managed: false
    - kind: redis
      managed: true
    - kind: horizontalpodautoscaler
      managed: false
    - kind: route
      managed: true
    - kind: mirror
      managed: true
    - kind: monitoring
      managed: false
    - kind: tls
      managed: true
    - kind: quay
      managed: true
    - kind: clairpostgres
      managed: false
  configBundleSecret: minio-config
```
#### Edit default config
```yaml
#Console -> Operators -> OperatorHub -> Installed Operators-> Red Hat Quay -> Quay Registry -> registry -> Config Editor Endpoint: registry-quay-config-editor-testvn-quay.apps.ocp.testvn.click (use account on registry-quay-config-editor-credentials-t4m6gc484b_
``` 
