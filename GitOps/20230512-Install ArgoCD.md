###Disable default argocd instance
```sh
oc patch subscription openshift-gitops-operator -n openshift-operators --type=merge -p='{"spec":{"config":{"env":[{"name":"DISABLE_DEFAULT_ARGOCD_INSTANCE","value":"true"}]}}}'
```
###Install GitOps Operator
```sh
Operators -> OperatorHub -> Red Hat OpenShift GitOps -> Install
``
###Install ArgoCD
Operators -> Installed -> Red Hat OpenShift GitOps -> ArgoCD -> 
```yml
---
kind: ArgoCD
apiVersion: argoproj.io/v1alpha1
metadata:
  name: argocd
  namespace: testvn-argocd
spec:
  nodePlacement: 
    nodeSelector: 
      node-role.kubernetes.io/infra: ""
    tolerations:
    - effect: NoSchedule
      operator: Equal
      key: node-function
      value: infra
  controller:
    resources:
      limits:
        cpu: 1000m
        memory: 1048Mi
      requests:
        cpu: 250m
        memory: 512Mi
  ha:
    enabled: false
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  rbac:
    defaultPolicy: 'role:admin'
    policy: |
      g, system:cluster-admins, role:admin
    scopes: '[groups]'
  redis:
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 250m
        memory: 128Mi
  repo:
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi
  resourceExclusions: |
    - apiGroups:
      - tekton.dev
      clusters:
      - '*'
      kinds:
      - TaskRun
      - PipelineRun        
  server:
    host: argocd.apps.ocp.testvn.click
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 125m
        memory: 128Mi
    insecure: true
    route:
      enabled: true
      tls:
        termination: edge
        insecureEdgeTerminationPolicy: Redirect
  sso:
    dex:
      openShiftOAuth: true
      resources:
        limits:
          cpu: 500m
          memory: 256Mi
        requests:
          cpu: 250m
          memory: 128Mi
    provider: dex
```

###Allow argocd access namespace
```sh
oc label namespace <namespace> argocd.argoproj.io/managed-by=<instance_name>
```
###Create argocd secret
```yml
kind: Secret
apiVersion: v1
metadata:
  name: in-cluster
  namespace: testvn-argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
  annotations:
    managed-by: argocd.argoproj.io
data:
  config: eyJ0bHNDbGllbn1iaW5zZWN1cmUiOmZhbHNlfX0=
  name: aW4tY1lcg==
  namespaces: dGVzdHZuLWFy14td29yZHByZXNz
  server: aHR0cHM6Ly9rd1ZmF1bHQuc3Zj
type: Opaque
```
###Create argocd repo  (same create repo on ArgoCD UI)
```yml
kind: Secret
apiVersion: v1
metadata:
  name: repo-2264682546
  namespace: testvn-argocd
  labels:
    argocd.argoproj.io/secret-type: repository
  annotations:
    managed-by: argocd.argoproj.io
data:
  password: Z2xwYXQtdnp6dnlqdHlrSEJESDdjU2d6ZUM=
  project: YXJnb2Nk
  type: Z2l0
  url: aHR0cHM6Ly9naXRsYWIudGVzdHZuLmNsaWNrL2Rldm9wcy9hcmdvY2QuZ2l0
  username: bm90LXVzZWQ=
type: Opaque
```
###Create Project  (same create project on ArgoCD UI)
```yml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: testvn-wordpress
  namespace: testvn-argocd
spec:
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
  destinations:
    - name: in-cluster
      namespace: testvn-wordpress
      server: 'https://kubernetes.default.svc'
  sourceRepos:
    - 'https://gitlab.testvn.click/devops/argocd.git'
```
###Create Application (same create application on ArgoCD UI)
```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: testvn-wordpress
  namespace: testvn-argocd
spec:
  destination:
    namespace: testvn-wordpress
    server: 'https://kubernetes.default.svc'
  project: testvn-wordpress
  source:
    path: wordpress
    repoURL: 'https://gitlab.testvn.click/devops/argocd.git'
    targetRevision: HEAD
  syncPolicy:
    automated:
    prune: true
    selfHeal: true
```


