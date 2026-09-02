# validated-pattern-aro-hcp-cluster-config

Org GitOps overlay for ARO HCP clusters. Argo CD syncs **this** repo; the operator baseline still comes from [`rh-mobb/validated-pattern-aro-hcp`](https://github.com/rh-mobb/validated-pattern-aro-hcp) via a Kustomize remote base.

Put tenant-specific desired state here (Entra group `cluster-admin`, extra Applications). Do not put it in the public installer repo.

## Layout

```
base/                         # org additions (today: one ClusterRoleBinding)
overlays/public/              # Public API clusters
overlays/private/             # Private API clusters
```

Each overlay includes `github.com/rh-mobb/validated-pattern-aro-hcp//gitops/overlays/<name>?ref=main` plus `base/`.

## Bootstrap

From the installer checkout, after `make cluster.<name>.kubeconfig` (and external-auth):

```bash
GITOPS_REPO=https://github.com/rh-mobb/validated-pattern-aro-hcp-cluster-config.git \
GITOPS_SOURCE_ROOT=overlays \
  make cluster.<name>.bootstrap
```

`GITOPS_OVERLAY` / `api_visibility` still pick `public` vs `private`. Changing repo later is another bootstrap (or `oc apply` of Application `cluster-config`).

If this GitHub repo is **private**, add it as a repository credential in OpenShift GitOps. The installer base is public.

## Cluster-admin group

[`base/cluster-admin-group.yaml`](base/cluster-admin-group.yaml) binds Entra security group **Cloud Services SSA Team - APAC** (`0ef73eda-7c79-4ae1-a848-9cfe87b10f87`) as OpenShift `cluster-admin`.

Add people in Entra, not with more YAML. The Entra app must emit group object IDs (`groupMembershipClaims=SecurityGroup`); `make cluster.<name>.external-auth` in the installer sets that.

The signed-in deployer is still bound as a **User** by external-auth (`entra-cluster-admin`) so bootstrap works before Argo is healthy.
