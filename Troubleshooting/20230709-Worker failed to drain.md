#### Check
```yaml
C:\Users\hoanb>oc get -o yaml clusteroperator machine-config
status:
  conditions:
  - lastTransitionTime: "2023-07-08T18:49:34Z"
    message: 'Failed to resync 4.12.17 because: error during syncRequiredMachineConfigPools:
      [timed out waiting for the condition, error pool worker is not ready, retrying.
      Status: (pool degraded: true total: 5, ready 0, updated: 0, unavailable: 1)]'
    reason: RequiredPoolsFailed
    status: "True"
    type: Degraded
  - lastTransitionTime: "2023-07-08T17:39:22Z"
    message: One or more machine config pools are degraded, please see `oc get mcp`
      for further details and resolve before upgrading
    reason: DegradedPool
    status: "False"
    type: Upgradeable
  extension:
    master: all 3 nodes are at latest configuration rendered-master-33aba53e4581bea756f16df327a5cb99
    worker: 'pool is degraded because nodes fail with "1 nodes are reporting degraded
      status on sync": "Node ocp-f944j-worker-dc28k is reporting: \"failed to drain
      node: ocp-f944j-worker-dc28k after 1 hour. Please see machine-config-controller
      logs for more information\""'

C:\Users\hoanb>oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-33aba53e4581bea756f16df327a5cb99   True      False      False      3              3
  3                     0                      98d
worker   rendered-worker-f7ab4dce950f7f84abfa880eff1da0aa   False     True       False      5              1
  1                     0                      98d

[root@lab-quay vault-helm]# oc logs -f -n openshift-machine-config-operator machine-config-controller-55b7b6c4bc-p95gt -c machine-config-controller
E0709 08:31:53.282530       1 drain_controller.go:110] error when evicting pods/"vault-0" -n "testvn-vault" (will retry after 5s): Cannot evict pod as it would violate the pod's disruption budget.
```
#### Solution
```yaml
oc delete pods/vault-0 -n testvn-vault
```
