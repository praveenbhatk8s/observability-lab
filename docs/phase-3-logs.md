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

None. Loki, the Alloy log pipeline, the datasource and the dashboard came up
on the first sync.

## What I learned

- **Labels are the index, not the content.** Loki indexes only the stream
  labels and stores the log lines compressed. A handful of low-cardinality
  labels (`namespace`, `app`, `container`) keeps it cheap; anything with many
  values, like a log level or a trace ID, belongs in the line or in structured
  metadata.
- **One label scheme for all signals.** Using the same `namespace`, `pod`,
  `container`, `app` and `cluster` labels as the metrics is what lets Grafana
  move from a metric spike to the logs of the same pod.
- **Where you read logs is a trade-off.** Tailing through the Kubernetes API
  keeps Alloy a single Deployment with no host access, but every line goes
  through the API server. At scale, a DaemonSet reading `/var/log/pods` on each
  node is the usual choice.
- **Kubernetes events are logs too.** Collecting them puts scheduling failures,
  failed image pulls and OOM kills in the same place as application logs, with
  their own retention instead of Kubernetes' one-hour default.
- **Single-binary Loki is enough for a lab.** The same chart scales out to
  separate read, write and backend paths with object storage when volume
  demands it; the storage schema (`tsdb`, `v13`) stays the same.
