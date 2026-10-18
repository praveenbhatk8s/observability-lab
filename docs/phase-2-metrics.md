# Phase 2: Metrics with Mimir and Alloy

## Goal

Cluster metrics flowing into Mimir and visible in Grafana:

```
kubelet / cAdvisor ─┐
kube-state-metrics ─┤
Mimir, Alloy itself ┼──> Alloy ──remote_write──> Mimir <──PromQL── Grafana
annotated pods ─────┘
```

## What was added

| File | Purpose |
|---|---|
| `apps/mimir.yaml`, `platform/mimir/` | Mimir in monolithic mode: StatefulSet, Service, config |
| `apps/alloy.yaml`, `platform/alloy/values.yaml` | Alloy collector and its scrape config |
| `apps/kube-state-metrics.yaml`, `platform/kube-state-metrics/values.yaml` | Object-state metrics |
| `apps/grafana-provisioning.yaml`, `platform/grafana/provisioning/` | Mimir datasource for Grafana |
| `apps/dashboards.yaml`, `dashboards/` | First dashboard: Cluster overview |

## Design decisions

- **Mimir in monolithic mode, from plain manifests.** The `mimir-distributed`
  chart runs every component as its own workload, which is the production
  shape but costs several GB. `-target=all` runs the same components in one
  process. Same storage format, same APIs; scaling out later means changing
  the target per workload.
- **Filesystem storage on a 5Gi volume**, 7-day retention. Object storage is a
  stretch goal.
- **Alloy as a single Deployment.** It finds targets through the Kubernetes API,
  so one replica covers the whole cluster.
- **Kubelet and cAdvisor through the API server proxy**
  (`/api/v1/nodes/<node>/proxy/metrics/...`), so Alloy needs only RBAC, not
  network access to each node.
- **Pod annotations for app metrics.** Any pod with `prometheus.io/scrape: "true"`
  and `prometheus.io/port` is scraped. The demo services use this in Phase 4.
- **Mimir metric names start with `cortex_`**, a leftover from the project Mimir
  was forked from.

## How to verify

```bash
kubectl -n argocd get app mimir alloy kube-state-metrics grafana-provisioning dashboards
kubectl -n observability get pods
```

Alloy's UI shows every component and whether it is healthy:

```bash
kubectl -n observability port-forward deploy/alloy 12345:12345
```

Open http://localhost:12345.

In Grafana: Explore, datasource Mimir, query `up`. Every target should be `1`.
Then open Dashboards, Platform, Cluster overview.

## What I learned

_Fill in after doing the phase._
