#### Install Minio Operator on OperatorHub
```sh
Console -> Operators -> OperatorHub -> Minio Operator -> Install (Default namespace)
```
#### Setup route for Minio Opeator
```yaml
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: minio-operator-console-route
  namespace: openshift-operators
spec:
  host: minio-operator.testvn.click
  path: /
  to:
    kind: Service
    name: console
    weight: 100
  port:
    targetPort: http
  tls:
    termination: edge
    certificate: |
    key: |
    caCertificate: |
  wildcardPolicy: None
```
#### Create role and rolebinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-role-minio
rules:
  - verbs:
      - create
    apiGroups:
      - ''
    resources:
      - namespaces
  - verbs:
      - get
    apiGroups:
      - ''
    resources:
      - resourcequotas
  - verbs:
      - delete
    apiGroups:
      - ''
    resources:
      - persistentvolumeclaims
---    
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: role-binding-minio
  namespace: openshift-operators
subjects:
- kind: ServiceAccount
  name: minio-operator
  namespace: openshift-operators
roleRef:
  kind: ClusterRole
  name: cluster-role-minio
  apiGroup: rbac.authorization.k8s.io
```
#### Create SCC
 oc get namespace testvn-minio -o=jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.supplemental-groups}{"\n"}'
```yaml
allowHostPorts: true
priority: null
requiredDropCapabilities: []
allowPrivilegedContainer: true
runAsUser:
  type: MustRunAs
  uid: 1001050000
users:
  - 'system:serviceaccount:testvn-minio:testvn-sa'
  - 'system:serviceaccount:testvn-minio:default'
allowHostDirVolumePlugin: true
allowHostIPC: true
seLinuxContext:
  type: MustRunAs
readOnlyRootFilesystem: false
metadata:
  name: scc-minio
fsGroup:
  type: MustRunAs
groups: []
kind: SecurityContextConstraints
defaultAddCapabilities: []
supplementalGroups:
  type: RunAsAny
volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - hostPath
  - persistentVolumeClaim
  - projected
  - secret
allowHostPID: true
allowHostNetwork: true
allowPrivilegeEscalation: true
apiVersion: security.openshift.io/v1
allowedCapabilities: []
```
#### Login URL https://minio-operator.testvn.click/ with Token
```yaml
[root@lab-quay ~]# oc get secret/minio-operator-token-vf5hq -n openshift-operators -o jsonpath="{.data.token}" | base64 --decode
eyJhbGciOiJSUzI1NiIsImtpZCI6ImJNeUhoZWtRanhBZUxPTXdub2Q5V3VjMTd6d2pad25ORTI5OGIyZGRoTjQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2N
```
#### Create Tenants
```yaml
#Setup with UI
Tenants -> Create Tenant
                        -> Setup
                              Name: minio
                              Namespace: testvn-minio
                              Storage Class: thin-csi
                              Number of Servers: 3
                              Drivers per Server: 2
                              Total Size: 1024
                              Erasure Code Parity: Ec4
                        -> Configure
                              Expose MinIO Service: On
                              Expose Console Service: On
                              Set Custom Domains Console: s3console.testvn.click
                              Set Custom Domains MinOO: s3.testvn.click
                              Security Context:
                                Run As User: 1001050000
                                Run As Group: 1001050000
                                FsGroup: 1001050000
                                Do not run as Root: On
                        -> Configure
                              TLS: Off
```
```yaml
#Setup with CLI
metadata:
  name: minio
  namespace: testvn-minio
scheduler:
  name: ""
spec:
  certConfig:
    commonName: '*.minio-hl.testvn-minio.svc.cluster.local'
    dnsNames:
    - minio-pool-0-{0...2}.minio-hl.testvn-minio.svc.cluster.local
    organizationName:
    - system:nodes
  configuration:
    name: minio-env-configuration
  credsSecret:
    name: minio-secret
  exposeServices:
    console: true
    minio: true
  features:
    domains:
      console: https://s3console.testvn.click
      minio:
      - https://s3.testvn.click
  image: minio/minio:RELEASE.2023-06-23T20-26-00Z
  imagePullPolicy: IfNotPresent
  imagePullSecret: {}
  mountPath: /export
  podManagementPolicy: Parallel
  pools:
  - affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
            - key: v1.min.io/tenant
              operator: In
              values:
              - minio
            - key: v1.min.io/pool
              operator: In
              values:
              - pool-1
          topologyKey: kubernetes.io/hostname
    name: pool-0
    resources: {}
    runtimeClassName: ""
    securityContext:
      fsGroup: 1001050000
      fsGroupChangePolicy: Always
      runAsGroup: 1001050000
      runAsNonRoot: true
      runAsUser: 1001050000
    servers: 3
    volumeClaimTemplate:
      metadata:
        creationTimestamp: null
        name: data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: "10737418240"
        storageClassName: thin-csi
      status: {}
    volumesPerServer: 2
  requestAutoCert: false
  serviceAccountName: minio-sa
  users:
  - name: minio-user-0
```
#### Create Route
```yaml
---
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: s3
  namespace: testvn-minio
spec:
  host: s3.testvn.click
  to:
    kind: Service
    name: minio
    weight: 100
  port:
    targetPort: http-minio
  tls:
    termination: edge
  wildcardPolicy: None
  ---
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: s3console
  namespace: testvn-minio
spec:
  host: s3console.testvn.click
  to:
    kind: Service
    name: minio-console
    weight: 100
  port:
    targetPort: http-console
  tls:
    termination: edge
  wildcardPolicy: None
```
