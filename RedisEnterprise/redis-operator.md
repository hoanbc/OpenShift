#### File SCC
```yml
allowHostPorts: true
priority: null
requiredDropCapabilities: []
allowPrivilegedContainer: true
runAsUser:
  type: MustRunAs
  uid: 1000
users:
  - 'system:serviceaccount:testvn-redis:redis'
allowHostDirVolumePlugin: true
allowHostIPC: true
seLinuxContext:
  type: MustRunAs
readOnlyRootFilesystem: false
metadata:
  name: scc-redis
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
#### File deploy
```yml
apiVersion: redis.redis.opstreelabs.in/v1beta1
kind: RedisCluster
metadata:
  name: redis-cluster
  namespace: testvn-redis
spec:
  persistenceEnabled: true
  serviceAccountName: redis
  redisLeader:
    affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: role
                  operator: In
                  values:
                    - leader
            topologyKey: kubernetes.io/hostname
    livenessProbe:
      failureThreshold: 3
      initialDelaySeconds: 1
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    readinessProbe:
      failureThreshold: 3
      initialDelaySeconds: 1
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
  kubernetesConfig:
    image: 'quay.io/opstree/redis:v7.0.5'
    imagePullPolicy: IfNotPresent
    updateStrategy: {}
    resources:
      limits:
        cpu: 300m
        memory: 512Mi
      requests:
        cpu: 100m
        memory: 128Mi
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  redisFollower:
    affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
                - key: role
                  operator: In
                  values:
                    - follower
            topologyKey: kubernetes.io/hostname
    livenessProbe:
      failureThreshold: 3
      initialDelaySeconds: 1
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    readinessProbe:
      failureThreshold: 3
      initialDelaySeconds: 1
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
  redisExporter:
    enabled: true
    image: 'quay.io/opstree/redis-exporter:v1.44.0'
    imagePullPolicy: IfNotPresent
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 100m
        memory: 128Mi
  clusterVersion: v7
  clusterSize: 3
  storage:
    volumeClaimTemplate:
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
        storageClassName: thin-csi
```
