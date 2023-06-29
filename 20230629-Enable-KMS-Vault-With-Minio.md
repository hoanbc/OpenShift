#### Enable AppRole Authentication
```sh
vault auth enable approle
```
#### Create a Policy for KES
```sh
#Create a policy with necessary capabilities for KES to use when accessing Vault. Select the tab corresponding to the KV engine used for storing KES secrets:
#Vault Engine V1
path "kv/*" {
      capabilities = [ "create", "read", "delete" ]
}

#Vault Engine V2
path "kv/data/*" {
      capabilities = [ "create", "read"]
}
path "kv/metadata/*" {
      capabilities = [ "list", "delete"]
}
```
#### Write the policy to Vault using vault policy write kes-policy kes-policy.hcl, Create an AppRole for KES and assign it the created policy
```sh
vault write    auth/approle/role/kes-role token_num_uses=0 secret_id_num_uses=0 period=5m
vault write    auth/approle/role/kes-role policies=kes-policy
```
#### Retrieve the AppRole ID and Secret
```sh
vault read     auth/approle/role/kes-role/role-id  #1
vault write -f auth/approle/role/kes-role/secret-id #2
```
### Enable Encryption for tenants Minio
```sh
URL: https://minio-operator.testvn.click -> Operator -> Tenants -> minio -> Encryption -> Save
              KMS: vault
                Endpoint: https://vault.testvn.click
                Engine: v2
              App Role:
                Engine: v2
                AppRole ID: #1
                AppRole Secret: #2
              SecurityContext for KES: Run As User/Group/FsGroup is 1001050000
```
### Enable Encryption for bucket Minio
```sh
URL: https://s3console.testvn.click -> Administrator -> Encryption -> Create Key -> Key Name:KMS-vault -> Save -> Administrator -> Bucket -> Your bucket -> Summary -> Edit Encryption -> Encryption Type: KMS -> KMS Key ID: KMS-vault ->Save
```
