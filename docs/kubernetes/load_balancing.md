---
tags:
  - MetalLB
  - Kubernetes
  - Networking
---

# Load Balancing

[TOC]

# Overview

This document details the network load-balancer implementation for the homelab kubernetes clusters.

## MetalLB

MetalLB is responsible for allocating IP addresses to `LoadBalancer` type services in the homelab to make them accessible from outside the cluster.

MetalLB is running in Layer 2 mode using ARP to make service IP addresses available to the rest of the home network. One of the master nodes in the cluster is responsible for advertising the services.

## Available and Assigned Address Pools

| CIDR | In Use | Assigned Cluster |
|------| ------ | ---------------- |
| 10.5.0.0/24 | :white_check_mark: | lab-dev |
| 10.6.0.0/24 | :white_check_mark: | lab-prod |

## Router Configuration

The homelab Asus router must be configured with a static route using the following syntax:

| Network/Host IP | Netmask | Gateway |
| --------------- | ------- | ------- |
| `<Subnet Address>` | `<Network Mask>` | `<IP address of a master node in the cluster>` |

## Deployment

MetalLB is deployed using FluxCD as part of the infrastructure stack on each cluster using Helm.

- The same base deployment is standardized across all clusters.
- Each cluster has a unique `IpAddressPool` and `L2Advertisement` applied via kustomization and Fluxcd during deployment.

## MetalLB Troubleshooting

### Failed to Call Webhook During Re-install

```
IPAddressPool/metallb-system/primary dry-run failed (InternalError): Internal error occurred: failed calling webhook "ipaddresspoolvalidationwebhook.metallb.io": failed to call webhook: Post "https://metallb-webhook-service.metallb-system.svc:443/validate-metallb-io-v1beta1-ipaddresspool?timeout=10s": service "metallb-webhook-service" not found...
```

Check to see if the following CRD exists. This CRD is not deleted when the MetalLB namespace is deleted because it is not namespace specific. This can lead to the error above when MetalLB is redeployed with a new operator.

```bash
kubectl get ValidatingWebhookConfiguration

NAME                            WEBHOOKS   AGE
cert-manager-webhook            1          26d
longhorn-webhook-validator      1          172d
metallb-webhook-configuration   8          297d
```

Delete the old webhook CRD

```bash
kubectl delete ValidatingWebhookConfiguration metallb-webhook-configuration
```