# fluxcd

FluxCD GitOps repo for the devops-training AKS clusters. Infrastructure
(AKS, Gateway, cert-manager, external-dns, Flux itself) lives in
[devops-training-project](https://github.com/mmirceta/devops-training-project);
this repo holds what Flux applies on top of it.

## Layout

```
base/<app>/                  Environment-independent manifests
<env>/kustomization.yaml     Everything Flux applies on that cluster (lists apps)
<env>/<app>/                 Kustomize overlay: base + env specifics (hostnames, replicas, ...)
```

Environments are folders, not branches: every cluster reads `main`, each from
its own `<env>` path. Add an app to an environment by creating its overlay and
listing it in `<env>/kustomization.yaml`; promote a change by editing the
target environment's overlay in a PR.

## Clusters

| Cluster | Path |
|---|---|
| aks-dev-training | `dev` |

## Headlamp (web UI)

Headlamp with the Flux plugin runs in the `headlamp` namespace, without
Ingress. Dev AKS is private, so reach it through the bastion:

```bash
# on the bastion VM
kubectl -n headlamp create token headlamp-user          # login token, valid 1h
kubectl -n headlamp port-forward --address 0.0.0.0 svc/headlamp 8080:80

# locally, second terminal
az network bastion tunnel --name bas-dev-training --resource-group rg-devops-training-dev \
  --target-resource-id <vm-bastion-dev resource id> --resource-port 8080 --port 8080
```

Open http://localhost:8080 and paste the token. `headlamp-user` can read
everything except Secrets and can reconcile / suspend / resume Flux objects.

## Useful commands

```bash
flux get sources git
flux get kustomizations
flux reconcile kustomization flux-system --with-source   # don't wait for the interval
kubectl kustomize dev                                     # render what dev gets
```
