```yml
  kind: "Service"
  apiVersion: "v1"
  metadata:
    name: "external-airbyte-service"
  spec:
    ports:
      -
        name: "airbyte"
        protocol: "TCP"
        port: 8000
        targetPort: 8000 
        nodePort: 0
  selector: {} 
---
  kind: "Endpoints"
  apiVersion: "v1"
  metadata:
    name: "external-airbyte-service" 
  subsets: 
    -
      addresses:
        -
          ip: "10.252.16.40" 
      ports:
        -
          port: 8000 
          name: "airbyte"
          
```
