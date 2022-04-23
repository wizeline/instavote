# Exporting Sync Manifests

In order to transfer the curret repository let's make a checklist of the sources
that are needed to re-create the cluster:

## Flux Resources

| Path                                                          | CRD            | Decription                               |
| :---                                                          | :----:         |                                     ---: |
| clusters/staging/instavote-gitrepository.yaml                 | GitRepository  | Source Reposytory for Kustomizations     |
| clusters/staging/helm-charts-helmrepository.yaml              | HelmRepository | Source Repository for HelmCharts         |
| clusters/staging/redis-staging-kustomization.yaml             | Kustomization  | Kustomization for Redis                  |
| clusters/staging/vote-staging-kustomization.yaml              | Kustomization  | Kustomization for Vote (Python)          |
| clusters/staging/worker-staging-kustomization.yaml            | Kustomization  | Kustomization for Worker (Java)          |
| clusters/staging/db-helmrelease.yaml                          | HelmRelease    | Postgres Release                         |
| clusters/staging/result-staging-helmrelease.yaml              | HelmRelease    | Result application Helm Chart            |
| clusters/staging/github-instavote-staging-alert-provider.yaml | Provider       | Connector to GitHub Instavote Repository |
| clusters/staging/redis-staging-alert.yaml                     | Alert          | Alert consumer for Alert Provider        |
| clusters/staging/vote-staging-alert.yaml                      | Alert          | Alert consumer for Alert Provider        |
| clusters/staging/worker-staging-alert.yaml                    | Alert          | Alert consumer for Alert Provider        |

Just make sure they are preent in your `flux-infra` repository for later usage.

## Secrets

### github-token

This is consumed by `./clusters/staging/github-instavote-staging-alert-provider.yaml` just make sure these values are
not checked-in in the reposutory as they'd expose the credentials. You can manually exporte them and re-upload them to
the new namespace when requierd in furter steps.
