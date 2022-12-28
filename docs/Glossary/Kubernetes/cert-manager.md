---
tags:
  - Kubernetes
  - CNCF
  - certificates
  - x509
---

# Cert-manager

## Installation

To perform a default installation of cert-manager[^1] one can perform the following command:

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.1/cert-manager.yaml
```

### SelfSigned ClusterIssuer

In order to setup a self-signed cluster-global issuer, apply the following manifest:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
```

### SelfSigned CA Certificate

In order to setup a self-signed ca certificate, apply the following manifests:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nb-annotator
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-selfsigned-ca
  namespace: nb-annotator
spec:
  isCA: true
  commonName: my-selfsigned-ca
  secretName: root-secret
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-ca-issuer
  namespace: nb-annotator
spec:
  ca:
    secretName: root-secret
```

[^1]: <https://cert-manager.io/docs/installation/>
