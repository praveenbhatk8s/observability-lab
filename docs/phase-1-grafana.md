# Phase 1: Grafana and the GitOps wiring

## Goal

A Grafana instance at http://grafana.localhost, deployed by Argo CD from this
repo, with its admin login coming from Vault and every datasource and dashboard
provisioned as code.

## What was added

| File | Purpose |
|---|---|
| `apps/observability-base.yaml` | Argo CD Application for `platform/base` |
| `apps/grafana.yaml` | Argo CD Application for the Grafana chart, values from this repo |
| `platform/base/namespace.yaml` | The `observability` namespace |
| `platform/base/grafana-admin.yaml` | ExternalSecret: Grafana admin login from Vault |
| `platform/grafana/values.yaml` | Grafana configuration |

The homelab repo has one bridge Application, `apps/observability-lab.yaml`,
that points Argo CD at this repo's `apps/` folder.

## Design decisions

- **Multi-source Application.** The chart comes from the Grafana Helm repo and
  the values from this repo (`$values/...`), so values stay in Git without a
  wrapper chart.
- **No persistence.** Grafana runs without a volume. Datasources and dashboards
  are ConfigMaps with the labels `grafana_datasource: "1"` and
  `grafana_dashboard: "1"`; the chart's sidecars load them. Restarting Grafana
  loses nothing that matters.
- **Admin password from Vault**, never in Git.
- **Community chart.** The `grafana/grafana` chart is deprecated; Grafana now
  publishes it from `grafana-community`. The values carried over unchanged.

## How to verify

```bash
kubectl -n argocd get app observability-base grafana
kubectl -n observability get externalsecret grafana-admin
kubectl -n observability get pods
```

Open http://grafana.localhost and sign in with the credentials stored in Vault.

## What I learned

_Fill in after doing the phase._
