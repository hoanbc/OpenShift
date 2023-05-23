## A Guide to GitOps and Secret Management with ArgoCD Operator and SOPS #

##### Create Generate Age Key
```sh
age-keygen -o age.agekey
Public key: age1helqcqsh9464r8chnwc2fzj8uv7vr5ntnsft0tn45v2xtz0hpfwq98cmsg
```
##### Create OpenShift Secret
```sh
cat age.agekey | oc create secret generic sops-age --namespace=openshift-gitops --from-file=keys.txt=/dev/stdin
```
##### ArgoCD with Custom Tooling (Installed Operators -> Argo CD -> Edit
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
      - mountPath: /.config/kustomize/plugin/viaduct.ai/v1/ksops
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
##### 1. Configure SOPS via .sops.yaml
 ```yml 
cat <<EOF > .sops.yaml
creation_rules:
  - path_regex: apps/.*\.sops\.ya?ml
    encrypted_regex: "^(data|stringData)$"
    age: age1helqcqsh9464r8chnwc2fzj8uv7vr5ntnsft0tn45v2xtz0hpfwq98cmsg
EOF
```
##### 2. Create a local Kubernetes Secret
```yml
cat <<EOF > secret.sops.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
EOF
```
##### 3. Encrypt with SOPS CLI:
```sh
sops --encrypt --in-place secret.sops.yaml
```
##### 4. Define KSOPS kustomize Generator:
```yml
cat <<EOF > secret-generator.yaml
apiVersion: viaduct.ai/v1
kind: ksops
metadata:
  # Specify a name
  name: example-secret-generator
files:
  - ./secret.sops.yaml
EOF
```
##### 5. Create the kustomization.yaml and Push all the changes to the Git repository. Read about kustomize plugins:
```yml
cat <<EOF > kustomization.yaml
generators:
  - ./secret-generator.yaml
EOF
```
##### 6. Create a new Argo application from the Argo console using your repository.
