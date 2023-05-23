###Administration -> Cluster Settings -> Configuration -> OAuth -> yaml
```yml
      - ldap:
        attributes:
          email:
            - mail
          id:
            - sAMAccountName
          name:
            - cn
          preferredUsername:
            - sAMAccountName
        bindDN: openshift
        bindPassword:
          name: ldap-bind-password-ggm7p
        ca:
          name: ldap-ca-5dkc8
        insecure: false
        url: >-
          ldaps://dc.lottefn.vn/OU=LOTTEFN,DC=lottefn,DC=vn?sAMAccountName?sub?(&(memberOf:1.2.840.113556.1.4.1941:=CN=N.HO.OpenShift,OU=LOTTEFN,DC=lottefn,DC=vn)(objectClass=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
      mappingMethod: claim
      name: Active Directory
      type: LDAP
```
