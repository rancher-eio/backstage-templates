# pgAdmin

pgAdmin is a web console for Postgres. EIO runs two instances, both read-only.

| Instance | Address | Database |
| --- | --- | --- |
| Comitiva | [comitiva.pgadmin.rancher.engineering](https://comitiva.pgadmin.rancher.engineering) | Comitiva |
| Observability | [metrics.pgadmin.rancher.engineering](https://metrics.pgadmin.rancher.engineering) | Mirror Metrics |

## Who can get in

Access is checked twice, and both checks use your GitHub account through
[Dex](https://dex.rancher.engineering).

1. **At the edge.** Before a request reaches pgAdmin, oauth2-proxy checks that
   you are signed in and that you are a member of the `rancher` GitHub org or
   the `rancher-eio:eio` team. Anyone else is turned away without reaching
   pgAdmin.
2. **In pgAdmin.** pgAdmin then runs its own GitHub sign in through Dex and
   checks that you are in one of the teams for that instance.

| Instance | Teams |
| --- | --- |
| Comitiva | `rancher-eio:eio`, `rancher:pgadmin-comitiva` |
| Observability | `rancher-eio:eio`, `rancher:pgadmin-metrics` |

GitHub is the only way to sign in. pgAdmin's own username and password login is
turned off.

Team membership is read when you sign in. If someone is removed from a team,
the edge check stops letting them in when their session expires, which takes
up to 7 days.

## What you can do

Each instance has one shared connection, already set up. It connects through
the database's read-only endpoint, which only reaches standby replicas, so
writes are rejected by Postgres.

| Instance | Connection | Database user |
| --- | --- | --- |
| Comitiva | Comitiva Database (Read-Only) | `comitiva` |
| Observability | Mirror Metrics Database (Read-Only) | `mirror` |

You are not asked for the database password. It is kept in 1Password and
handed to pgAdmin as a password file when pgAdmin starts.

## Getting access

Ask EIO to add you to the team for the instance you need.
