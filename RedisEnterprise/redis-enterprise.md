#### Create SCC
```yml
kind: SecurityContextConstraints
apiVersion: security.openshift.io/v1
metadata:
  name: redis-enterprise-scc
allowPrivilegedContainer: false
allowedCapabilities:
  - SYS_RESOURCE
runAsUser:
  type: MustRunAs
  uid: 1001
FSGroup:
  type: MustRunAs
  ranges: 1001,1001
seLinuxContext:
  type: RunAsAny
```
#### Deploy Redis Enterprise Operator
-> Namespace -> Operator -> Operator Hub -> Redis Enterprise Operator -> Install (production)

#### Add SCC to service account
```sh
oc adm policy add-scc-to-user redis-enterprise-scc system:serviceaccount:testvn-redisenterprisecluster:redis-enterprise-operator
oc adm policy add-scc-to-user redis-enterprise-scc system:serviceaccount:testvn-redisenterprisecluster:rec
```
#### Add Cluster
```yml
apiVersion: app.redislabs.com/v1
kind: RedisEnterpriseCluster
metadata:
  name: rec
spec:
  redisEnterpriseNodeResources:
    limits:
      cpu: 2000m
      memory: 4Gi
    requests:
      cpu: 200m
      memory: 1Gi
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - values:
                - rec
              key: redis.io/cluster
              operator: In
            - values:
                - node
              key: redis.io/role
              operator: In
        topologyKey: kubernetes.io/hostname
  persistentSpec:
    enabled: true
    storageClassName: thin-csi 
    volumeSize: 10Gi
  redisEnterpriseImageSpec:
    imagePullPolicy: IfNotPresent
    repository: registry.connect.redhat.com/redislabs/redis-enterprise
  bootstrapperImageSpec:
    repository: registry.connect.redhat.com/redislabs/redis-enterprise-operator
  redisEnterpriseServicesRiggerImageSpec:
    repository: registry.connect.redhat.com/redislabs/services-manager
  nodes: 3
  uiServiceType: ClusterIP
  username: devops@testvn.click
```
#### Add external connect
```yml
kind: Service
apiVersion: v1
metadata:
  annotations:
    metallb.universe.tf/loadBalancerIPs: '10.200.12.229'
    metallb.universe.tf/address-pool: 'prod-ip-pools'
  name: redis1-expose
  namespace: testvn-redisenterprisecluster
spec:
  ipFamilies:
    - IPv4
  ports:
    - name: redis
      protocol: TCP
      port: 12084
      targetPort: 12084
  internalTrafficPolicy: Cluster
  type: LoadBalancer
  ipFamilyPolicy: SingleStack
  sessionAffinity: None
  selector:
    redis.io/bdb-4: '4'
```