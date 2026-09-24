# How self-service works

Self-service means you ask for a repository, or access to a secret, by filling
in a form instead of by asking a person. This page explains what happens after
you press the button, and what each piece is for.

## What you see

1. You open Backstage and pick a template. There is one for creating a
   repository, and one for giving a repository access to a shared secret.
2. You fill in the form and press Create.
3. Backstage opens a tracking issue for you in
   [rancherlabs/eio](https://github.com/rancherlabs/eio/issues). This is your
   single place to follow the request.
4. Backstage writes the YAML your request needs and opens a pull request in
   [internal-production](https://github.com/rancher-eio/internal-production/pulls).
5. The pull request is linked from your tracking issue, so a reviewer can click
   straight through to it.
6. EIO reviews and merges.
7. A few minutes after the merge, the resource exists in the cluster and a
   comment appears on your tracking issue saying so.

You do not need to know any of what follows. It is here so the team can support
it.

## What is behind it

### The image

Backstage is not used as it ships. We build our own image from
[rancher-eio/backstage](https://github.com/rancher-eio/backstage). It adds the
plugins we use and the tooling that renders these docs. There is no CI in that
repository, so the image is built and pushed by hand, and the tag is recorded in
`config.yaml`.

### Where it runs

The image is deployed by
[internal-production](https://github.com/rancher-eio/internal-production), like
everything else we run. The settings live in
[backstage-values.yaml](https://github.com/rancher-eio/internal-production/blob/main/manifests/backstage/resources/ConfigMap/backstage-values.yaml),
and the tag to deploy lives in
[backstage.yaml](https://github.com/rancher-eio/internal-production/blob/main/manifests/backstage/resources/HelmRelease/backstage.yaml).
That is the portal you log into.

### What fills the screen

An empty Backstage shows nothing. Everything you see in it comes from
[backstage-catalog](https://github.com/rancher-eio/backstage-catalog): the list
of services, these docs, and the templates themselves. Backstage is pointed at
that one repository and reads it on a loop, so anything merged there shows up in
the portal without a deploy.

### The templates

A template is a form plus a list of steps. These are the ones we have:

| Template | What it does |
| --- | --- |
| [Request secret access](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/secrets/request-secret-access.yaml) | Gives an existing repository access to a shared secret. |
| [Create a repository (direct)](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/github/create-repository-request-direct.yaml) | Creates a repository, with a pull request straight to internal-production. |
| [Create a repository](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/github/create-repository-request.yaml) | Creates a repository by handing the request to that organization's own `org` repository, so organization management stays in one place. |
| [Create a Docker Hub repository](https://github.com/rancher-eio/backstage-catalog/blob/main/templates/dockerhub/create-dockerhub-repo.yaml) | Opens an issue for EIO to action. There is nothing to generate for this one. |

They all follow the same four steps, in this order:

1. Open the tracking issue.
2. Write the YAML.
3. Open the pull request.
4. Write the pull request link back into the issue.

The order is deliberate. The issue has to exist first, because its number is
written into both the YAML and the pull request description. Step 4 is a
separate workflow,
[link-pr-to-issue.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/link-pr-to-issue.yaml),
because the pull request does not have a number until step 3 has run.

For secret requests there is a fifth step. It runs
[add-vault-role-and-policy.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/add-vault-role-and-policy.yaml),
which creates the Vault role and policy that let the repository read the secret.

### Telling you it worked

The YAML a template writes carries two labels with it:

```yaml
annotations:
  github.eio.rancher.engineering/requested-by: "your-username"
  github.eio.rancher.engineering/issue-number: "4452"
```

Two things run in the cluster watching for those. `k8s-event-exporter` notices
when one of these resources becomes ready, and hands it to
`eio-k8s-github-notifier`, which reads the issue number off the resource and
comments on that issue.

This is the part that closes the loop. Without it you would have to go and check
the cluster yourself to find out whether your request actually worked.

## The workflows

Some of this runs as a GitHub Actions workflow in
[internal-production](https://github.com/rancher-eio/internal-production/actions)
rather than in Backstage. That is where the Vault roles and policies get
created, and where the YAML lands, so the workflows sit next to the things they
change. Backstage triggers them and moves on. You can watch any of them run.

| Workflow | Triggered by | What it does |
| --- | --- | --- |
| [link-pr-to-issue.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/link-pr-to-issue.yaml) | Every template, after the pull request is opened | Adds a `Pull request` section to your tracking issue with a link to the pull request. |
| [add-vault-role-and-policy.yaml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/add-vault-role-and-policy.yaml) | Secret requests, and repository requests that ask for secrets | Creates the Vault role and policy the repository needs, then writes its own section into the tracking issue. |
| [add-to-project.yml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/add-to-project.yml) | Any issue or pull request being opened | Puts it on the [EIO board](https://github.com/orgs/rancherlabs/projects/32) so nothing gets lost. |
| [request-access-secret.yml](https://github.com/rancher-eio/internal-production/blob/main/.github/workflows/request-access-secret.yml) | Dispatch, with a repository name and a secret type | Opens a pull request granting secret access. The older route to the same result, kept for callers that still use it. |

A workflow triggered this way has to already be on `main`. If it is not, the
trigger fails quietly and nothing happens, which is worth knowing when a request
looks like it half worked.

## The repositories

| Repository | What it holds |
| --- | --- |
| [rancher-eio/backstage](https://github.com/rancher-eio/backstage) | The image we build and run. |
| [rancher-eio/internal-production](https://github.com/rancher-eio/internal-production) | The deployment, the generated YAML, and the workflows. |
| [rancher-eio/backstage-catalog](https://github.com/rancher-eio/backstage-catalog) | The templates, these docs, and the service list. |
| [rancherlabs/eio](https://github.com/rancherlabs/eio/issues) | Every tracking issue. |
| `<org>/org` | Organization and team management, kept out of internal-production on purpose. |
