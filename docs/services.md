# Services

EIO services you can sign into. They all sit behind Dex, so you sign in once.

## Metrics and logs

| Service | Address | What it is |
| --- | --- | --- |
| Grafana | [grafana.rancher.engineering](https://grafana.rancher.engineering) | Dashboards for the clusters and services EIO operates. |
| Grafana (Engineering) | [metrics.rancher.engineering](https://metrics.rancher.engineering) | Dashboards for engineering teams. |
| Logs | [logs.rancher.engineering](https://logs.rancher.engineering) | OpenSearch Dashboards. Search logs from anything running in cluster. |
| Prometheus | [prometheus.rancher.engineering](https://prometheus.rancher.engineering) | Raw metrics and PromQL. |
| Alertmanager | [alertmanager.rancher.engineering](https://alertmanager.rancher.engineering) | Firing alerts, silences and routing. |
| Grafana (kube-prometheus) | [grafana.prometheus.rancher.engineering](https://grafana.prometheus.rancher.engineering) | The Grafana bundled with the monitoring stack, for cluster level dashboards. |

## Platform

| Service | Address | What it is |
| --- | --- | --- |
| Backstage | [backstage.rancher.engineering](https://backstage.rancher.engineering) | This portal. Service catalog, docs and self-service requests. |
| Vault | [vault.rancher.engineering](https://vault.rancher.engineering) | Secrets. See [Vault secrets](vault-secrets.md) for how to read them from a workflow. |
| Dex | [dex.rancher.engineering](https://dex.rancher.engineering) | Single sign on. Everything here authenticates through it. |
| Goldilocks | [goldilocks.rancher.engineering](https://goldilocks.rancher.engineering) | Suggested CPU and memory requests for your workloads. |

## Databases

| Service | Address | What it is |
| --- | --- | --- |
| pgAdmin (Comitiva) | [comitiva.pgadmin.rancher.engineering](https://comitiva.pgadmin.rancher.engineering) | Postgres console for Comitiva. |
| pgAdmin (Observability) | [metrics.pgadmin.rancher.engineering](https://metrics.pgadmin.rancher.engineering) | Postgres console for the observability stack. |

## Missing something?

This is what is exposed today, not everything EIO runs. If something you use is
not listed, ask EIO.
