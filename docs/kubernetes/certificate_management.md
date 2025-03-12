---
tags:
  - Kubernetes
  - TLS
---

# Certificate Management

[TOC]

## Overview

This sections provides and overview of certifiacte management on the home lab kubernetes clusters.

## Cert Manager

Cert Manager is used to communicate with a self hosted HashiCorp Vault CA and issue certificates as needed during application deployment. Cert manager is also responsible for rotating certificates during expiration.

### Cert Manager and Issuer Deployment

Cert manager is deployed in a standarized way across all clusters using FluxCD with Kustomize. A `ClusterIssuer` CRD is created on each cluster and configured to authenticate to HashiCorp Vault for issuing certificates.

## Issuing Certificates

Certificates are issued using the following `Certificate` CRD during application deployment:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: podinfo-com
  namespace: uat-test-app
spec:
  secretName: podinfo-tls
  issuerRef:
    name: vault-issuer
    kind: ClusterIssuer
  commonName: www.podinfo-uat.lab.com
  dnsNames:
  - www.podinfo-uat.lab.com
```
