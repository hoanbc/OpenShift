## A Guide to GitOps and Secret Management with ArgoCD Operator and SOPS #

#### Create Generate Age Key
```sh
age-keygen -o age.agekey
Public key: age1helqcqsh9464r8chnwc2fzj8uv7vr5ntnsft0tn45v2xtz0hpfwq98cmsg
```
#### Create OpenShift Secret
```sh
cat age.agekey | oc create secret generic sops-age --namespace=openshift-gitops --from-file=keys.txt=/dev/stdin
```
#### ArgoCD with Custom Tooling (Installed Operators -> Argo CD -> Edit
```yml
  repo:
    env:
      - name: XDG_CONFIG_HOME
        value: /.config
      - name: SOPS_AGE_KEY_FILE
        value: /.config/sops/age/key.txt
    initContainers:
      - args:
          - >-
            echo "Installing KSOPS..."; cp ksops /custom-tools/; cp
            $GOPATH/bin/kustomize /custom-tools/; echo "Done.";
        command:
          - /bin/sh
          - '-c'
        image: 'viaductoss/ksops:v3.0.2'
        name: install-ksops
        volumeMounts:
          - mountPath: /custom-tools
            name: custom-tools
    volumeMounts:
      - mountPath: /usr/local/bin/kustomize
        name: custom-tools
        subPath: kustomize
      - mountPath: /.config/kustomize/plugin/viaduct.ai/v1/ksops/ksops
        name: custom-tools
        subPath: ksops
      - mountPath: /.config/sops/age
        name: sops-age
    volumes:
      - emptyDir: {}
        name: custom-tools
      - name: sops-age
        secret:
          secretName: sops-age
  kustomizeBuildOptions: '--enable-alpha-plugins'
  ```
  
  
