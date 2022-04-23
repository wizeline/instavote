# Deploying with HelmRelease

With the HelmRepository avilable is now time to deploy our first Helm
application for our database.


## Generating the values file to override configurations

This database needs a bit of cusmotization in order to be discoverable and able
to accept annonymous connections, here the settings needed:

```sh
cd flux-infra
mkdir -p values/staging
cat > values/staging/values.db.yaml << EOF
settings:
  authMethod: trust
service:
  name: db
EOF
```

## Generate the Release

Now we can generate the HelmRelease object via:

```sh
flux create helmrelease db \
  --source=HelmRepository/helm-charts \
  --chart=postgres \
  --values=./values/staging/values.db.yaml \
  --target-namespace=instavote \
  --export > ./clusters/staging/db-helmrelease.yaml
```

Time to commit and publish the changes so the Reconciliation can happen:

```sh
git add clusters/staging
git commit -am "feat: Add HelmRelease control"
git push origin HEAD:refs/heads/main
```

## Verify the deployment

You can trigger a reconciliation and verify the release status

```sh
flux reconcile source git flux-system

# Verify the HelmRelease
flux get helmreleases db
```

## Verify the worker deployment

Now that the HelmRelease has been applied we should be able to see the conenction
```sh
kubectl logs --selector app=worker
Waiting for db
Connected to db
Watching vote queue
```
