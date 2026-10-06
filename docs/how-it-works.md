# How self-service works

Backstage is the front door to the EIO platform. You ask for what you need there,
and automation does the rest. Today you can request repositories and secret
access. More will be added as we automate it.

```
 Backstage  ──►  tracking issue  ──►  pull request  ──►  merge  ──►  cluster
                        ▲                                               │
                        └────────────────── notifier ───────────────────┘
```

The tracking issue in [rancherlabs/eio](https://github.com/rancherlabs/eio/issues)
is the single place to follow a request. Every pull request it produces is
linked back into that issue, and once the resource is live in the cluster the
notifier comments there to say so. Expect a few minutes between the merge and
the comment.

## The parts

| Piece | Repository |
| --- | --- |
| The Backstage image, built and pushed by hand | [rancher-eio/backstage](https://github.com/rancher-eio/backstage) |
| Its deployment, the generated YAML, the workflows | [rancher-eio/internal-production](https://github.com/rancher-eio/internal-production) |
| The templates, these docs, the service list | [rancher-eio/backstage-catalog](https://github.com/rancher-eio/backstage-catalog) |
| Tracking issues | [rancherlabs/eio](https://github.com/rancherlabs/eio/issues) |
| Organization and team management | `<org>/org` |

Backstage re-reads backstage-catalog on a loop, so anything merged there reaches
the portal without a deploy. The image is the exception: it has no CI, so a new
one has to be built by hand and its tag recorded in `config.yaml`.

## The templates

A template is a form plus a list of steps.

| Template | What it does |
| --- | --- |
| [Request secret access](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/secrets/request-secret-access.yaml) | Gives an existing repository access to a shared secret. |
| [Create a repository (direct)](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/github/create-repository-request-direct.yaml) | Creates a repository, with a pull request straight to internal-production. |
| [Create a repository](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/github/create-repository-request.yaml) | The same, but handed to that organization's own `org` repository, so organization management stays in one place. |
| [Create a Docker Hub repository](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/dockerhub/create-dockerhub-repo.yaml) | Opens an issue for EIO. Nothing to generate for this one. |

All of them open the tracking issue first, then write the YAML and submit the
pull request. The issue comes first because its number is written into both.

## The workflows

Some of this runs as a GitHub Actions workflow in
[internal-production](https://github.com/rancher-eio/internal-production/actions)
rather than in Backstage. That is where the Vault roles and policies get
created, and where the YAML lands, so the workflows sit next to the things they
change. Backstage triggers them and moves on. You can watch any of them run.

| Workflow | Triggered by | What it does |
| --- | --- | --- |
| [link-pr-to-issue.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/link-pr-to-issue.yaml) | Every template, once the pull request is open | Adds a `Pull request` section to the tracking issue. |
| [add-vault-role-and-policy.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/add-vault-role-and-policy.yaml) | Requests that involve secrets | Creates the Vault role and policy in both clusters, and links both pull requests from the issue. |
| [add-to-project.yml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/add-to-project.yml) | Any issue or pull request opening | Puts it on the [EIO board](https://github.com/orgs/rancherlabs/projects/32). |
| [request-access-secret.yml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/request-access-secret.yml) | Dispatch, with a repository name and a secret type | Opens a pull request granting secret access. An older route to the same result. |

A workflow triggered this way has to already be on `main`. If it is not, the
trigger fails quietly, which is worth knowing when a request looks like it half
worked.

## How the notifier knows

Generated YAML carries the requester and the issue number with it:

```yaml
annotations:
  github.eio.rancher.engineering/requested-by: "username"
  github.eio.rancher.engineering/issue-number: "4452"
```

`k8s-event-exporter` notices the resource becoming ready and hands it to
`eio-k8s-github-notifier`, which reads the issue number off it and comments
there. Without this last step a request would have to be checked against the
cluster by hand.
