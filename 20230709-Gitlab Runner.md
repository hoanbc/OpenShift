#### Install DevWorkspace Operator and Gitlab Runner
```yaml
OpenShift Console -> Operators -> OperatorHub -> DevWorkspace Operator -> Default Install
OpenShift Console -> Operators -> OperatorHub -> Gitlab Runner -> Install with namespace custom (testvn-gitlab)
```
#### Create Gitlab Runner
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-runner-secret
  namespace: testvn-gitlab
type: Opaque
stringData:
  runner-registration-token: XXXXXXXXXXX
---
apiVersion: apps.gitlab.com/v1beta2
kind: Runner
metadata:
 name: gitlab-runner
spec:
 gitlabUrl: https://gitlab.testvn.click
 buildImage: alpine
 token: gitlab-runner-secret
 tags: openshift
```
