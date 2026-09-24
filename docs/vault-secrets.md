# Vault secrets

Shared credentials are stored in Vault under a path unique to your repository.
Nothing else can read them, and you do not need to copy them into GitHub
repository secrets.

Read them in a workflow with
[read-vault-secrets](https://github.com/rancher-eio/read-vault-secrets).

```yaml
permissions:
  contents: read
  id-token: write

steps:
  - uses: rancher-eio/read-vault-secrets@main
    with:
      secrets: |
        <the lines from your section below>
```

Your secrets become readable once the pull request on your request is merged,
and the sync runs about a minute later. Before that you will get a permission
error.

If a workflow cannot read a path listed here, the repository is probably
missing its Vault role. Open an issue with EIO rather than retrying.

## Docker Hub

```
secret/data/github/repo/${{ github.repository }}/dockerhub/rancher/credentials username | DOCKERHUB_USERNAME ;
secret/data/github/repo/${{ github.repository }}/dockerhub/rancher/credentials password | DOCKERHUB_PASSWORD
```

## Rancher Prime Registry

```
secret/data/github/repo/${{ github.repository }}/rancher-prime-registry/credentials registry | PRIME_REGISTRY ;
secret/data/github/repo/${{ github.repository }}/rancher-prime-registry/credentials username | PRIME_REGISTRY_USERNAME ;
secret/data/github/repo/${{ github.repository }}/rancher-prime-registry/credentials password | PRIME_REGISTRY_PASSWORD
```

Two different registry accounts are published to this path: a Rancher one and a
Harvester one. Your repository is granted exactly one of them, never both.

They matter because their push scope differs. The Rancher account cannot push
into `/harvester/` images, and a repository wired to the wrong one fails with
`insufficient_scope` at push time rather than at read time. The read lines above
are the same either way, so the failure does not show up until you push.

If your repository publishes Harvester images, say so when you request access.

## Rancher Prime Staging Registry

```
secret/data/github/repo/${{ github.repository }}/rancher-prime-stg-registry/credentials registry | STAGING_REGISTRY ;
secret/data/github/repo/${{ github.repository }}/rancher-prime-stg-registry/credentials username | STAGING_REGISTRY_USERNAME ;
secret/data/github/repo/${{ github.repository }}/rancher-prime-stg-registry/credentials password | STAGING_REGISTRY_PASSWORD
```

The same Rancher and Harvester split applies here. See the note above.

## Requesting access

Use the
[Request Secret Access](https://backstage.rancher.engineering/create/templates/default/request-secret-access)
template for a repository that already exists, or pick the secrets on the
Secrets page when you create a new repository.

Need something that is not listed? Describe it in the "Not listed?" box on that
template and EIO will set it up.
