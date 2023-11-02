# Helm Chart for IR Nucleus


This directory contains everything from building docker images to Helm Chart Values.


All values files  are synced with environments , we defined in ArgoCD definitions.

- values-dev.yaml [eks-dev]
- values-qa.yaml [eks-qa]
- values-prod.yaml [eks-prod]


## How to build a new image tag

Using the actions tab , and run the pipeline Docker Build ECR with the below inputs
- image tag
- environment

Once the pipline is completed , new image for this repo is published to AWS ECR based on environment.

Next step is to update the image tag in values yaml for specific environment.

Once the tag is updated ,argoCD will sync the changes and deploy new version to the specific cluster.



## Chart Details


Replica counts
```yaml
replicaCount: 1

```

Image tag
```yaml
image:
  repository: 238070276932.dkr.ecr.us-east-1.amazonaws.com/ir-nucleus-web
  pullPolicy: Always
  # Overrides the image tag whose default is the chart appVersion.
  tag: latest

```

Load secrets from vault  
```yaml
secrets:
  enabled: true
  spec:
    refreshInterval: '15s'
    secretStoreRef:
      name: vault-backend
      kind: ClusterSecretStore
    target:
      name: ir-nucleus-env
    data:
      - secretKey: DATABASE_URL
        remoteRef:
          key: secret/ir-nucleus-dev
          property: DATABASE_URL


```

Ingress / External URL for application
```yaml
ingress:
  enabled: true
  className: 'nginx'
  annotations:
    kubernetes.io/tls-acme: 'true'
    cert-manager.io/cluster-issuer: 'letsencrypt-prod'
    nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'
  hosts:
    - host: ir-nucleus.k8s.dev.innovationrefunds.com


```



> **Warning:**
> All application are fetched via secrets from vault and mount as environment variable in application.
