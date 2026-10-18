# Phase 3: Logs with Loki

## Goal

Every container's logs, plus Kubernetes events, searchable in Grafana with the
same labels as the metrics from Phase 2:

```
pod logs (Kubernetes API) ─┐
Kubernetes events ─────────┴──> Alloy ──push──> Loki <──LogQL── Grafana
```

## What was added

| File | Purpose |
|---|---|
| `apps/loki.yaml`, `platform/loki/values.yaml` | Loki 3.6, single binary, 5Gi volume, 7-day retention |
| `platform/alloy/values.yaml` (log section) | Pod log tailing, event collection, push to Loki |
| `platform/grafana/provisioning/datasource-loki.yaml` | Loki datasource |
| `dashboards/logs-overview.json` | Log volume, error rate, log browser, events |

## Design decisions

- **Single-binary Loki.** The chart's other modes split Loki into read, write
  and backend paths with object storage. One process is enough at this volume.
- **Logs read through the Kubernetes API** (`loki.source.kubernetes`), not from
  node files. Alloy stays a single Deployment with no host mounts. The
  trade-off: every log line passes through the API server, which wouldn't scale
  to a large cluster, where a DaemonSet reading `/var/log/pods` is the norm.
- **Same label names as metrics** (`namespace`, `pod`, `container`, `app`,
  `cluster`). That's what makes switching between metrics and logs in Grafana
  work.
- **Few index labels.** Log level goes into structured metadata, not a label,
  so it doesn't multiply Loki's streams.
- **Kubernetes events as logs**, under `job="kubernetes-events"`, so OOM kills
  and failed pulls show up next to the application logs.

## How to verify

```bash
kubectl -n argocd get app loki alloy grafana-provisioning dashboards
kubectl -n observability get pods
```

In Grafana:
- Explore, datasource Loki: `{namespace="observability"}`.
- Explore, `{job="kubernetes-events"}`.
- Dashboards, Platform, Logs overview.
- Drilldown, Logs: Grafana's query-less log browser, which uses the volume and
  pattern features enabled in `values.yaml`.

## Problems hit

_None yet._

## What I learned

_Fill in after doing the phase._
