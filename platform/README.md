# platform

What every app on Heap runs on. A broken change here breaks everyone's app, so say in the PR what you tested and tag `@recursecenter/heap-admins`.

Each directory holds one Argo CD Application: `application.yaml` pins a chart, `values.yaml` configures it, and `manifests/` (where present) holds plain YAML deployed alongside it.

| Component | Chart | Wave | Notes |
|---|---|---|---|
| `argocd` | argo-cd 10.18.0 (Argo CD 3.6.1) | -3 | Manages itself. Manual sync; press sync and watch. |
| `cert-manager` | cert-manager v1.21.3 | -2 | ClusterIssuer `letsencrypt-prod`, Cloudflare DNS01 |
| `sealed-secrets` | sealed-secrets 2.22.0 (controller 0.41.2) | -2 | Back up the controller key off the cluster |
| `csi-driver-nfs` | csi-driver-nfs 4.13.6 | -2 | Default StorageClass `heap-nfs` on broome:/data/k8s |
| `kuberay-operator` | kuberay-operator 1.7.2 | -2 | Operator and CRDs only |
| `monitoring` | kube-prometheus-stack 93.3.1 | -1 | Prometheus and Grafana; scrapes the hosts' node_exporter on 9100 |
| `nvidia-device-plugin` | nvidia-device-plugin 0.20.1 | 0 | Runs on `gpu=true` nodes |

Lower waves sync first, and the root app waits for each wave to go healthy before starting the next. That wait depends on the Application health check in `argocd/values.yaml`. Anything that ships CRDs sits in a wave before anything that uses them; objects that use another component's CRD carry `SkipDryRunOnMissingResource=true`.

## Adding a component

Copy a directory, change the chart, version and namespace, and pick a wave. `bootstrap/root.yaml` picks up any `platform/*/application.yaml` on its own.

## Upgrading a chart

Bump `targetRevision`, then run `scripts/render.sh` on `main` and on your branch and diff the two. CI posts the same diff on the PR.
