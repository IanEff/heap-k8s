# heap-k8s

Everything that runs inside Heap's Kubernetes cluster, as files Argo CD syncs from `main`. Merge a directory under `apps/` and it's running a few minutes later; delete it and it's gone.

Any Recurser can open, review and approve a PR here, `platform/` included. Nothing in the repo stops you from changing the parts everyone depends on. The layout and the READMEs say where those parts are.

If you read two sections, read **Adding an app** and **What lives where**.

## Adding an app

1. Copy `apps/_template` to `apps/<name>`. Your app runs in namespace `app-<name>`.
2. Replace `deployment.yaml` and `service.yaml` with your own manifests and list them in `kustomization.yaml`.
3. Open a PR. CI renders your directory and checks it against the Kubernetes 1.36 schemas.
4. Once it merges, watch it come up at https://argocd.recurse.cloud.

`apps/README.md` covers volumes, GPUs, Ingress and secrets.

## What lives where

| Directory | What it holds | Who it affects when it breaks |
|---|---|---|
| `apps/` | One directory per community app | That app |
| `platform/` | Argo CD, cert-manager, sealed-secrets, csi-driver-nfs, the NVIDIA device plugin, the KubeRay operator, Prometheus and Grafana | Every app on the cluster |
| `projects/` | The two Argo CD projects: `platform` may create cluster-scoped objects, `community` may not | Every app on the cluster |
| `bootstrap/` | The root app, the ApplicationSet over `apps/*`, and the script that installs Argo CD on a fresh cluster | Everything, on the next rebuild |

## What this repo doesn't own

The hosts, k3s and Cilium belong to [recursecenter/cluster-config](https://github.com/recursecenter/cluster-config), which installs them with Ansible. Argo CD never touches k3s or Cilium. Cilium is also the ingress controller (IngressClass `cilium`), so there is no Traefik here.

Every persistent volume lives on broome's `/data/k8s` over NFS. When broome or its disk is down, every pod with a volume hangs.

## Checking a change locally

```sh
scripts/validate.sh
```

It needs `helm`, `kustomize`, `yq` and `kubeconform`. `scripts/render.sh <dir>` writes out everything Argo CD would apply, one file per Application, if you want to read the YAML a chart bump produces.
