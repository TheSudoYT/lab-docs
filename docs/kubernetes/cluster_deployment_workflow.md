# Cluster Deployment Workflow

1. virtual machines deployed
2. Static route added to router directing traffic destined for the cluster CIDR range to one of the master node ip addresses.
3. Cluster formed.
4. Cluster directory created in github repo
5. Vault mounts configured and secrets created in Vault
    - Mount k8s auth method
    - Create a role on auth method for PKI and a role for kv secrets - cert manager and vso get seprate roles
    - pki role gets vault wide CA policy
    - vso role gets kv secrets policy for either dev or prod secrets mount
6. Flux-cd installed on cluster
7. kustomizations reconciled and cluster configured
8. Health checks and alerts to determine cluster readiness 

