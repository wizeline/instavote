# The Need for Multi-Tenancy

## Current Implementation

```mermaid
    stateDiagram-v2

    state instavote {
        direction LR
        vote/
        worker/
        result/
        state deploy/ {
            direction LR
            deploy/vote: vote
            deploy/redis: redis
            deploy/worker: worker
            state charts/ {
                result
            }
        }
    }

    state flux-infra {
        direction LR
        state clusters {
            direction LR
            state dev {
                dev_gotk.yaml: flux-system/gotk.yaml
            }
            state staging {
                direction LR
                    gitrepository : gitrepository.yaml
                    kustomization: kustomization.yaml
                state flux-system {
                    gotk.yaml
                }
            }
            state production {
                production_gotk.yaml: flux-system/gotk.yaml
            }
        }
    }
```

Problems with this setup:

- Both Dev & Ops teams have access to the `flux-infra` resources:
  - Cluster configs
  - All namespaces
  - All environments
- All code in one place `instavote`
- Onboard of new projects is hard

## Design for Multi-Tenancy


```mermaid

    stateDiagram-v2

    state instavote {
        direction LR
        vote
        worker
        result
    }

    state deploy {
        direction LR
        state flux {
            direction LR
            gitrepository : gitrepository.yaml
            kustomization: kustomization.yaml

        }
        state kustomize {
            kustomize_vote: vote
            kustomize_redis: redis
            kustomize_worker: worker
        }
        state helm/charts {
            direction LR
            helm_charts_result: result
        }
    }

    state fleet {
        direction LR
state clusters {
            direction LR
            dev/
            state staging/flux-system {
                direction LR
                gotk.yaml
            }
            production/
        }
        state projects {
            projects_instavote: instavote
            facebooc
        }
    }

```

Impvoments:

- CI/CD cycles on separate cycles
  - One change to the application code does not trigger a deployment automatically
  - One change to the deployment code does no trigger a build in the app
- Application specific codes contained in the namespace (tenant)
- `fleet` repository only accesible to SRE and Platform engineers


## Futher Reading:

* https://github.com/fluxcd/flux2-multi-tenancy
