### GitOps with Vault
##### 1) Create Vault Configuration Secret
```yml
kind: Secret
apiVersion: v1
metadata:
  name: vault-configuration
  namespace: testvn-argocd
data:
  AVP_AUTH_TYPE: dG9rZW4=
  AVP_TYPE: dmF1bHQ=
  SOPS_AGE_KEY_FILE: Ly5jb25maWcvc29wcy9hZ2Uva2V5LnR4dA==
  VAULT_ADDR: aHR0cHM6Ly92YXVsdC50ZXN0dm4uY2xpY2s=
  VAULT_TOKEN: aHZzLnNXcm01TTRmVEFCcHdiNkZ4Yng4cjNJQQ==
  XDG_CONFIG_HOME: Ly5jb25maWc=
type: Opaque
```
##### 2) Install GitOps with argocd-vault-plugin (custom-tools)
```yml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: testvn-argocd
spec:
  nodePlacement:
    nodeSelector:
      node-role.kubernetes.io/infra: ''
    tolerations:
      - effect: NoSchedule
        key: node-function
        operator: Equal
        value: infra
  server:
    autoscale:
      enabled: false
    grpc:
      ingress:
        enabled: false
    host: argocd.apps.ocp.testvn.click
    ingress:
      enabled: false
    insecure: true
    resources:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 125m
        memory: 128Mi
    route:
      enabled: true
      tls:
        insecureEdgeTerminationPolicy: Redirect
        termination: edge
    service:
      type: ''
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
  rbac:
    defaultPolicy: 'role:admin'
    policy: |
      g, system:cluster-admins, role:admin
      g, system:grp-devops, role:admin
    scopes: '[groups]'
  repo:
    envFrom:
      - secretRef:
          name: vault-configuration
    initContainers:
      - command:
          - sh
          - '-c'
          - cp /tools/argocd-vault-plugin /custom-tools/
        image: 'docker.io/hoanbc/vault-plugin:latest'
        name: download-tools
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - mountPath: /usr/local/bin/argocd-vault-plugin
        name: custom-tools
        subPath: argocd-vault-plugin
    volumes:
      - emptyDir: {}
        name: custom-tools
  #kustomizeBuildOptions: '--enable-alpha-plugins'
  configManagementPlugins: |
    - name: argocd-vault-plugin
      init:
        command: [sh, -c]
        args: ["helm dependency build"]
      generate:
        command: ["sh", "-c"]
        args: ["helm template $ARGOCD_APP_NAME -n $ARGOCD_APP_NAMESPACE ${ARGOCD_ENV_helm_args} . --include-crds | argocd-vault-plugin generate -"]
```
##### 3) Create a Secret in Vault (Example)
![image](https://github.com/hoanbc/OpenShift/assets/26288807/28972387-c607-45b5-849d-8196886daaa9)
![image](https://github.com/hoanbc/OpenShift/assets/26288807/69c09a8c-92e1-4dbb-ac27-5a330b2916c8)

##### 4) Create Helm Chart
```yml
#Note: with .dockerconfigjson key need convert to base64 (<secretpull> -> base64: PHNlY3JldHB1bGw+ )
---
apiVersion: v1
kind: Secret
metadata:
  name: testvn-pull-secret
  labels:
    project: testvn
    environment: uat
    managed-by: helm
  annotations:
    avp.kubernetes.io/path: secret/data/argocd-secrets
data:
  .dockerconfigjson: PHNlY3JldHB1bGw+
type: kubernetes.io/dockerconfigjson
---
#Note: with other key don't need convert to base64 (<certificate> )
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: uat-test123-public-route
  labels:
    project: testvn
    environment: uat
    managed-by: helm
    type: testvn-public
  annotations:
    avp.kubernetes.io/path: secret/data/ssl
    haproxy.router.openshift.io/set-forwarded-headers: append
    haproxy.router.openshift.io/timeout: 5000ms
spec:
  host: hostform-los-uat.lottefinance.vn
  path: /
  wildcardPolicy: None
  port:
    targetPort: app-port
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
    key: |
      <key>
    certificate: |
      <certificate>
    caCertificate: |
      <caCertificate>
  to:
    kind: Service
    name:  uat-test123-public-svc
    weight: 100
```
##### 5) Create Applications ArgoCD
```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: test1112
spec:
  destination:
    name: ''
    namespace: testvn-demo
    server: 'https://kubernetes.default.svc'
  source:
  plugin:
    name: argocd-vault-plugin
    env:
      - name: helm_args
        value: '-f values123.yaml'
    path: hostform-los-uat-fe-123
    repoURL: 'https://gitlab.testvn.click/devops/argocd.git'
    targetRevision: HEAD
  sources: []
  project: wordpress
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
```
