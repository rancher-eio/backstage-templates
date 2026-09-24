# Services

What EIO runs. The [internal](#internal) ones sit behind Dex, so you sign in
once. The [public](#public) ones take no sign in at all, they answer Rancher
installs out in the world.

## Internal

### Metrics and logs

| Service | Address | What it is |
| --- | --- | --- |
| Grafana | [grafana.rancher.engineering](https://grafana.rancher.engineering) | Dashboards for the clusters and services EIO operates. |
| Grafana (Engineering) | [metrics.rancher.engineering](https://metrics.rancher.engineering) | Dashboards for engineering teams. |
| Logs | [logs.rancher.engineering](https://logs.rancher.engineering) | OpenSearch Dashboards. Search logs from anything running in cluster. |
| Prometheus | [prometheus.rancher.engineering](https://prometheus.rancher.engineering) | Raw metrics and PromQL. |
| Alertmanager | [alertmanager.rancher.engineering](https://alertmanager.rancher.engineering) | Firing alerts, silences and routing. |
| Grafana (kube-prometheus) | [grafana.prometheus.rancher.engineering](https://grafana.prometheus.rancher.engineering) | The Grafana bundled with the monitoring stack, for cluster level dashboards. |

### Platform

| Service | Address | What it is |
| --- | --- | --- |
| Backstage | [backstage.rancher.engineering](https://backstage.rancher.engineering) | This portal. Service catalog, docs and self-service requests. |
| Vault | [vault.rancher.engineering](https://vault.rancher.engineering) | Secrets. See [Vault secrets](vault-secrets.md) for how to read them from a workflow. |
| Dex | [dex.rancher.engineering](https://dex.rancher.engineering) | Single sign on. Everything here authenticates through it. |
| Goldilocks | [goldilocks.rancher.engineering](https://goldilocks.rancher.engineering) | Suggested CPU and memory requests for your workloads. |

### Databases

| Service | Address | What it is |
| --- | --- | --- |
| pgAdmin (Comitiva) | [comitiva.pgadmin.rancher.engineering](https://comitiva.pgadmin.rancher.engineering) | Postgres console for Comitiva. |
| pgAdmin (Observability) | [metrics.pgadmin.rancher.engineering](https://metrics.pgadmin.rancher.engineering) | Postgres console for the observability stack. |

### External cluster

The external cluster runs its own copies of these, separate from the ones above.

| Service | Address | What it is |
| --- | --- | --- |
| Dex (external) | [dex.rancher.systems](https://dex.rancher.systems) | Single sign on for the external cluster. |
| Goldilocks (external) | [goldilocks.rancher.systems](https://goldilocks.rancher.systems) | Request sizing for workloads on the external cluster. |

## Public

These are on the internet with no sign in, because installs everywhere call
them. They run on the external cluster.

Use the `rancher.systems` address. The `rancher.io` ones next to it are aliases
on the same ingress, same certificate, same backend, kept because released
products already call them. Point anything new at `rancher.systems`.

### Channel servers

They tell an install which release channel points at which version.

| Service | Address | Also answers on | What it is |
| --- | --- | --- | --- |
| Rancher | [rancher.channels.rancher.systems](https://rancher.channels.rancher.systems) | `update.rancher.io` | Release channels for Rancher. |
| RKE2 | [rke2.channels.rancher.systems](https://rke2.channels.rancher.systems) | `update.rke2.io` | Release channels for RKE2. |
| K3s | [k3s.channels.rancher.systems](https://k3s.channels.rancher.systems) | `update.k3s.io` | Release channels for K3s. |

### Upgrade responders

An install checks in anonymously, and gets told the latest version available.

They serve `/v1/checkupgrade` and nothing else, so the bare address below will
not load in a browser.

| Service | Address | Also answers on | What it is |
| --- | --- | --- | --- |
| Rancher Desktop | `desktop.version.rancher.systems` | `desktop.version.rancher.io` | Version checks from Rancher Desktop. |
| Epinio | `epinio.version.rancher.systems` | `epinio.version.rancher.io` | Version checks from Epinio. |
| Harvester | `harvester.version.rancher.systems` | `harvester-upgrade-responder.rancher.io` | Version checks from Harvester. |
| Longhorn | `longhorn.version.rancher.systems` | `longhorn-upgrade-responder.rancher.io` | Version checks from Longhorn. |
| NeuVector | `neuvector.version.rancher.systems` | `upgrades.neuvector-upgrade-responder.livestock.rancher.io` | Version checks from NeuVector. |
| RKE2 security | `rke2-security.version.rancher.systems` | `security-responder.version.rke2.io` | Security advisories for RKE2. |

## Missing something?

This is what is exposed today, not everything EIO runs. If something you use is
not listed, ask EIO.
