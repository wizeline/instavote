# Generate the Worker Deployment

Now lets generate a Kustomization for the Worker app written in Java

```sh
cd instavote
mkdir -p deploy/worker/{base,dev,staging}
```

Generate the deployment configuration:

```sh
kubectl create deployment worker \
  --image=schoolofdevops/worker:latest \
  --dry-run=client \
  -o yaml > deploy/worker/base/deployment.yaml
```

We don't need a Service files as we don't accept incomming connections.


Now generate the Kustomize base file

```sh
pushd deploy/worker/base &>/dev/null
  kustomize create --autodetect
popd &>/dev/null
```

And finally generate the layers for development and staging

```sh
# Generate the overlay configuration
for env in deploy/worker/{dev,staging}; do
  pushd ${env} &>/dev/null
    kustomize create --resources ../base
  popd &>/dev/null
done
```

We're done with the `instavote` repository, commit and submit your changes to the repository.


```sh
git add deploy
git commit -m "build: Add worker Kustomizations"
git push origin HEAD:refs/heads/main
```

## Add the Kustomize Object in the Cluster


Switch to the `flux-infra` repository and create the kustomization object. Note
that we're only generating the worker object for the `staging` environment.

```sh
flux create kustomization worker-staging \
    --source=instavote \
    --path="./deploy/worker/staging" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/worker.instavote" \
    --depends-on=redis-staging \
    --export > clusters/staging/worker-staging-kustomization.yaml
```

Commit and Promote your changes
```sh
git add clusters/staging/
git commit -am "feat: Add Worker Kustomization"
git push origin HEAD:refs/heads/main
```

Reconcile the flux system or just wait for it to take efect:

```sh
flux reconcile source git flux-system

# Watch for the new Kustomize object
flux get kustomizations worker-staging
```

Finally you can verify for the worker logs:
```sh
$ kubectl logs --selector app=worker
Waiting for db
Waiting for db
Waiting for db
```

We can now proceed to deply the DB with the Helm Controller:

```sh
flux create kustomization worker-staging \
    --source=instavote \
    --path="./deploy/worker/staging" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/worker.instavote" \
    --depends-on=redis-staging \
    --export > clusters/staging/worker-staging-kustomization.yaml
```
