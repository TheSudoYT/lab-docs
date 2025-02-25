# FluxCD

## Directory Structure

A method to the madness quote

Standardization

All applications and objects are deployed via the `base` directory as much as possible. Each cluster deploys the same configuration from the `base` kustomization and only applys a cluster specific kustomization overlay when required. This enables a standardized approach ensuring each cluster is configured as expected, but also provided flexibility when unique parameters and configurations are required. 

Dependencies

Some applications require CRDs to be created, but they cannot be created until the `HelmRelease` is created. This issue is addressed by creating 2 seperate kustomizations - 1 for the `HelmRelease` and any other objects required to the future CRDs to be created and 1 for the CRDs. The kustomization with dependencies can then utilize the `dependsOn` parameter to ensure the `HelmRelease` is created before the CRDs.

Examples if this in the lab are:

- A Cert Manager kustomization must install the Cert Manager operator before the CertIssuer CRD can be created
- A Traefik kustomization must install the Traefik operator before the IngressRoute for the traefik dashboard can be created
- A MetalLB kustomization must install the MetalLB operator and speakers before IpAddressPools and L2Advertisements can be created
