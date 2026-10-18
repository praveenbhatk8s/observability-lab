# observability-lab

Grafana LGTM (Loki, Grafana, Tempo, Mimir) plus Pyroscope on Kubernetes,
deployed with Argo CD, with a small set of instrumented Go services to observe.

The goal is one Grafana where metrics, logs, traces and profiles for the same
request link to each other.

## Status

| Phase | Topic | Status |
|---|---|---|
| 0 | Repo skeleton and resource check | Done |
| 1 | Grafana and GitOps wiring | Done |
| 2 | Metrics: Mimir, and Alloy as the collector | Done |
| 3 | Logs: Loki | Done |
| 4 | Demo services in Go, instrumented with OpenTelemetry | Next |
| 5 | Traces: Tempo, with trace, log and metric correlation | Planned |
| 6 | Profiles: Pyroscope, linked from traces | Planned |
| 7 | Dashboards, alerts and an SLO as code, plus k6 load | Planned |
| 8 | Write-up and architecture diagram | Planned |

Phase notes are in [`docs/`](docs/):
[1 Grafana](docs/phase-1-grafana.md) ·
[2 Metrics](docs/phase-2-metrics.md) ·
[3 Logs](docs/phase-3-logs.md)

## What's running so far

```
kubelet / cAdvisor ────┐
kube-state-metrics ────┤                ┌──> Mimir ───┐
pod logs, k8s events ──┼──> Alloy ──────┤             ├──> Grafana
Mimir, Alloy itself ───┘                └──> Loki  ───┘
```

| Endpoint | What |
|---|---|
| http://grafana.localhost | Grafana; dashboards in the Platform folder |
| `kubectl -n observability port-forward deploy/alloy 12345:12345` | Alloy's pipeline UI |

## Runs on

The [homelab](https://github.com/praveenbhatk8s/homelab) cluster: k3d on an
Apple Silicon Mac, Argo CD, Vault and External Secrets Operator. The homelab's
Argo CD picks up every Application in this repo's `apps/` folder.

## Layout

```
apps/        Argo CD Applications, one per component
platform/    Configuration per component (Helm values, base manifests)
services/    Go source for the demo services
deploy/      Kubernetes manifests for the demo services
dashboards/  Grafana dashboards as code
alerts/      Alert and recording rules
load/        k6 load scripts
docs/        Phase notes and architecture
```

## Versions

Charts come from two Helm repos: `grafana` (https://grafana.github.io/helm-charts)
and `grafana-community` (https://grafana-community.github.io/helm-charts),
where Grafana moved the Grafana and Tempo charts.


| Component | Helm chart | App version |
|---|---|---|
| Grafana | `grafana-community/grafana` 14.0.1 | 13.2.4 |
| Alloy | `grafana/alloy` 1.13.1 | 1.20.1 |
| kube-state-metrics | `prometheus-community/kube-state-metrics` 8.6.0 | |
| Mimir | plain manifests, image `grafana/mimir:3.2.1` (monolithic) | 3.2.1 |
| Loki | `grafana/loki` 7.3.0 | 3.6.12 |
| Tempo | `grafana-community/tempo` 3.2.0 | 3.1.0 |
| Pyroscope | `grafana/pyroscope` 2.4.0 | 2.4.0 |
