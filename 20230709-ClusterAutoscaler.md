#### Create MachineAutoScaler
```yml
apiVersion: autoscaling.openshift.io/v1beta1
kind: MachineAutoscaler
metadata:
  name: ocp-f944j-worker
  namespace: openshift-machine-api
spec:
  maxReplicas: 5
  minReplicas: 3
  scaleTargetRef:
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    name: ocp-f944j-worker
```
#### Create ClusterAutoscaler
```yaml
apiVersion: autoscaling.openshift.io/v1
kind: ClusterAutoscaler
metadata:
  name: default
spec:
  podPriorityThreshold: -10
  resourceLimits:
    cores:
      max: 128
      min: 8
    maxNodesTotal: 12
    memory:
      max: 512
      min: 16
  scaleDown:
    delayAfterAdd: 10m
    delayAfterDelete: 5m
    delayAfterFailure: 30s
    enabled: true
    unneededTime: 60s
```
#### Create nginx for test autoscale
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: nginx
  name: 'nginx'
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 50
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: container
          image: >-
            docker.io/nginxinc/nginx-unprivileged:latest
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            requests:
              memory: "1Gi"
              cpu: "1"
            limits:
              memory: "1Gi"
              cpu: "1"
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```
#### Monitor scaler
```yaml
watch -n 1 oc get machines -n openshift-machine-api
oc logs -f pod/cluster-autoscaler-default-55776cb8b8-mfk8f -n openshift-machine-api
```
