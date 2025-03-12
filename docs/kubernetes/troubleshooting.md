# Troubleshooting Kubernetes

[TOC]

# Overview

Common problems relate to kubernetes infrastructure and deployements and how to resolve them.

## FluxCD Kustomizations Failing when using CRDs

The following errors mean you must inform FluxCD kustmozations of dependencies so that FluxCD can deploy them in the correct order. An example of this is cert-manager. You must ensure that the cert-manager operator is installed before attempting to deploy the cert-issuer CRDs. This is because the `Issuer` and `ClusterIssuer` CRDs do not yet exist until the operator is installed. 

```bash
kubectl get kustomizations -n flux-system

NAME                     AGE    READY   STATUS
apps                     8d     False   IngressRouteTCP/uat-test-app/uat-test-app dry-run failed: no matches for kind "IngressRouteTCP" in version "traefik.io/v1alpha1"...
cert-issuer              26d    True    Applied revision: main@sha1:52aa57ed7cba7684dbf517405dab33237aa92e34
cert-manager             26d    True    Applied revision: main@sha1:52aa57ed7cba7684dbf517405dab33237aa92e34
flux-system              27d    True    Applied revision: main@sha1:52aa57ed7cba7684dbf517405dab33237aa92e34
metallb                  6d4h   False   IPAddressPool/metallb-system/primary dry-run failed: no matches for kind "IPAddressPool" in version "metallb.io/v1beta1"...
nfs                      26d    True    Applied revision: main@sha1:52aa57ed7cba7684dbf517405dab33237aa92e34
traefik                  7d3h   False   IngressRoute/traefik-v2/dashboard dry-run failed: no matches for kind "IngressRoute" in version "traefik.io/v1alpha1"...
vault-secrets-operator   26d    True    Applied revision: main@sha1:52aa57ed7cba7684dbf517405dab33237aa92e34
```

# Namespaces stuck in terminating 

When a namespace is stuck in terminating run the following command:

```bash
kubectl api-resources --verbs=list --namespaced -o name | while read -r resource; do
    kind=$(kubectl api-resources --namespaced --output wide | grep "^$resource" | awk '{print $1}')
    echo "Resource: $resource, Kind: $kind"
    kubectl get "$resource" -n vault-secrets-operator
done
```

The command will return the name and kind of resources stuck terminating within the namespace. Take those resources and edit the following command. providing the resources and namespace in question, to remove them:

```bash
kubectl -n vault-secrets-operator patch vaultconnection default -p '{"metadata":{"finalizers":null}}' --type=merge
```