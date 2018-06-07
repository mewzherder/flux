---
title: Setup Weave Flux Manually
menu_order: 20
---

# Detailed Description

The deployment installs Flux and its dependencies. First, change to
the directory with the examples configuration.

## Memcache

Flux uses memcache to cache docker registry requests.

```sh
kubectl create -f memcache-dep.yaml -f memcache-svc.yaml
```

## Flux deployment

You will need to create a secret in which Flux will store its SSH
key. The daemon won't start without this present.

The `flux` logs should show that it has now connected to the
repository and synchronised the cluster.

When using Kubernetes, this key is stored as a Kubernetes secret. You
can restart `flux` and it will continue to use the same key.

### 2. Specify a key to use

Create a Kubernetes Secret from a private key:

```
kubectl create secret generic flux-git-deploy --from-file=identity=/path/to/private_key
```

The Kubernetes deployment configuration file
[flux-deployment.yaml](../../deploy/flux-deployment.yaml) runs the
Flux daemon, but you'll need to edit it first, at least to supply your
own configuration repo (the `--git-repo` argument).

```sh
$EDITOR flux-deployment.yaml
kubectl create -f flux-deployment.yaml
```

### Note for Kubernetes >=1.6 with role-based access control (RBAC)

You will need to provide fluxd with a service account which can access
the namespaces you want to use Flux with. To do this, consult the
example service account given in
[flux-account.yaml](../../deploy/flux-account.yaml) (which
puts essentially no constraints on the account) and the
[RBAC documentation](https://kubernetes.io/docs/admin/authorization/rbac/),
and create a service account in whichever namespace you put fluxd
in. You may need to alter the `namespace: default` lines, if you adapt
the example.

You will need to explicitly tell fluxd to use that service account by
uncommenting and possible adapting the line `# serviceAccountName:
flux` in the file `fluxd-deployment.yaml` before applying it.