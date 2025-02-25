---
tags:
  - Traefik
  - Ingress
  - Kubernetes
  - Networking
---

# Ingress

[TOC]

# Overview

This section outlines the ingress controller(s) deployment, configuration, and use cases in the home lab.

## Traefik

Traefik-v2 is used as the primary and default ingress controller for the clusters.

### Treafik Deployment

Traefik is deployed using FluxCD as part of the infrastructure stack on each cluster using Helm.

- The same base deployment is standardized across all clusters.
- Each application defines an `Ingress` of class name`Traefik` and other related CRDs as needed within their kustomizations.

### Defining Ingresses

Each new ingress contains the following `annotations` as a minimum:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: uat-test-app
  namespace: uat-test-app
  annotations:
    spec.ingressClassName: traefik
    traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
    traefik.ingress.kubernetes.io/frontend-entry-points: https
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  rules:
    - host: podinfo-uat.lab.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: podinfo
                port:
                  number: 9898
  tls:
    - hosts:
        - podinfo-uat.lab.com
      secretName: podinfo-tls
```

### TLS Passthrough

When TLS passthrough is desired a CRD of kind `IngressRouteTCP` is used instead of an ingress. This permits TLS to be terminated at the application pod instead of at a Traefik ingress

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRouteTCP
metadata:
  name: uat-test-app
  namespace: uat-test-app
  annotations:
    traefik.ingress.kubernetes.io/router.tls.passthrough: "true"
spec:
  entryPoints:
    - websecure
  routes:
    - match: HostSNI(`podinfo-uat.lab.com`)
      services:
        - name: podinfo
          port: 9899
  tls:
    passthrough: true
```

