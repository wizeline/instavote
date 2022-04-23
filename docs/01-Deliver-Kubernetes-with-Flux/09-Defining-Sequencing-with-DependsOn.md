# Defining Sequencing with DependsOn

Kustmization can have an order declared for their deployments, in this case
we're are going to add a new Kustomization for `redis` and mark the dependency
in the `vote` kustomization

## Create a redis Kustomization

Generate the kustomize yaml object in `flux-infra` repository
```sh
# See the generated Outputs
flux create kustomization redis-dev \
    --source=instavote \
    --path="./deploy/redis" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/redis.instavote" \
    --export

# Save the contents to a file
flux create kustomization redis-dev \
    --source=instavote \
    --path="./deploy/redis" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/redis.instavote" \
    --export > clusters/dev/redis-dev-kustomization.yaml

# Commit the changes
git add clusters/dev/redis-dev-kustomization.yaml
git commit -am "chore: add redis kustomization in dev cluster"
git push origin HEAD:refs/heads/main
```

Now wait for the reconciliation to happen or manually reconcile the cluster

```
flux reconcile kustomization flux-system
```

### Mark the dependecy in the vote Deployment

Add the dependency to `vote-dev` kustomization
```sh
flux create kustomization vote-dev \
    --source=instavote \
    --path="./deploy/vote" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/vote.instavote" \
    --depends-on=redis-dev \
    --export > clusters/dev/vote-dev-kustomization.yaml

# Check for the differences
clusters/dev/vote-dev-kustomization.yaml

# Commit the changes
git add clusters/dev/vote-dev-kustomization.yaml
git commit -am "chore: add dependency to redis from vote"
git push origin HEAD:refs/heads/main
```
