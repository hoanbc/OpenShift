#### 1) Create configmap
vim cluster-monitoring-config.yaml
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |+
    enableUserWorkload: true
    alertmanagerMain:
      volumeClaimTemplate:
        metadata:
          name: vmware-alertmanager-claim
        spec:
          storageClassName: thin-csi
          resources:
            requests:
              storage: 10Gi
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    prometheusK8s:
      retention: 3d
      volumeClaimTemplate:
        metadata:
          name: vmware-prometheus-claim
        spec:
          storageClassName: thin-csi
          resources:
            requests:
              storage: 20Gi
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    grafana:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    k8sPrometheusAdapter:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    kubeStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    telemeterClient:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    openshiftStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    thanosQuerier:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
```
#### 2) Apply configmap
oc apply -f cluster-monitoring-config.yaml
