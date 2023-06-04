```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-workload-monitoring-config
  namespace: openshift-user-workload-monitoring
data:
  config.yaml: |
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    prometheus:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
      retention: 3d
      volumeClaimTemplate:
        metadata:
          name: vmware-prometheus-claim
        spec:
          storageClassName: thin-csi
          resources:
            requests:
              storage: 20Gi
    thanosruler:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-function
        operator: Equal
        value: infra
        effect: NoSchedule
    ```
