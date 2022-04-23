# Migrating the Deployment Code

Now we're going to migrathe the deployment code from `instavote` to its final
location on `instavote-deploy` to decouple the application form its delivery
code. For this we'll also need `fr-infra` to move the Sync Manifests.

Make sure the 3 repositories are prenset locally under this paths:

- instavote
- instavote-deploy
- flux-infra

```sh
$ ls
flux-infra       instavote        instavote-deploy
```

Now proceed with the migration:

## Migrating the Application Deployment code

This is the code related to the application

```sh
# Move the Application Charts
$ mv -v instavote/deploy/charts/result instavote-deploy/helm/charts/
instavote/deploy/charts/result -> instavote-deploy/helm/charts/result

# Move Application Kustomizations
$ mv -v instavote/deploy/{redis,vote,worker} instavote-deploy/kustomize
instavote/deploy/redis -> instavote-deploy/kustomize/redis
instavote/deploy/vote -> instavote-deploy/kustomize/vote
instavote/deploy/worker -> instavote-deploy/kustomize/worker
```

## Migrating the Flux Manifest

We'll be moving the flux Sync Manifests to `instavote-deploy/flux/base`

```sh
$ cp -v flux-infra/clusters/staging/*.yaml instavote-deploy/flux/base
flux-infra/clusters/staging/db-helmrelease.yaml -> instavote-deploy/flux/base/db-helmrelease.yaml
flux-infra/clusters/staging/github-instavote-staging-alert-provider.yaml -> instavote-deploy/flux/base/github-instavote-staging-alert-provider.yaml
flux-infra/clusters/staging/helm-charts-helmrepository.yaml -> instavote-deploy/flux/base/helm-charts-helmrepository.yaml
flux-infra/clusters/staging/instavote-gitrepository.yaml -> instavote-deploy/flux/base/instavote-gitrepository.yaml
flux-infra/clusters/staging/redis-staging-alert.yaml -> instavote-deploy/flux/base/redis-staging-alert.yaml
flux-infra/clusters/staging/redis-staging-kustomization.yaml -> instavote-deploy/flux/base/redis-staging-kustomization.yaml
flux-infra/clusters/staging/result-staging-helmrelease.yaml -> instavote-deploy/flux/base/result-staging-helmrelease.yaml
flux-infra/clusters/staging/vote-staging-alert.yaml -> instavote-deploy/flux/base/vote-staging-alert.yaml
flux-infra/clusters/staging/vote-staging-kustomization.yaml -> instavote-deploy/flux/base/vote-staging-kustomization.yaml
flux-infra/clusters/staging/worker-staging-alert.yaml -> instavote-deploy/flux/base/worker-staging-alert.yaml
flux-infra/clusters/staging/worker-staging-kustomization.yaml -> instavote-deploy/flux/base/worker-staging-kustomization.yaml
```

Add the Kustomization Overlay

```sh
pushd instavote-deploy/flux/base/
  kustomize create --autodetect
popd

# Verify the contets were created
$ head instavote-deploy/flux/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- db-helmrelease.yaml
- github-instavote-staging-alert-provider.yaml
- helm-charts-helmrepository.yaml
- instavote-gitrepository.yaml
- redis-staging-alert.yaml
- redis-staging-kustomization.yaml
- result-staging-helmrelease.yaml
```

### Update the Namespace

The Flux components will no longer belong to `flux-system` in this new tenant model
proceed to update the `namespace` property

```sh
sed -i '' 's/namespace: flux-system/namespace: instavote/g' flux/base/*.yaml
```

## Update the references to point to the new repositories

Now that we have the code in place, we need to update some values such as the
`GitRepository`, `Provider`, and `Kustomization` to the new structure

```sh
# Update the Repository Url references
cd instavote-deploy
find flux -name '*.yaml' -exec sed -i '' "s|${GITHUB_USER}/instavote|&-deploy|g" {} \;

# Update the paths to the new Kustomization
find flux -name '*kustomization.yaml' -exec  sed -i '' 's|./deploy/|./kustomize/|g' {} \;

# Update the paths fot the new Charts
find flux -name '*helmrelease.yaml' -exec  sed -i '' 's|./deploy/|./helm/|g' {} \;
```

## Commit and Publish the changes

Now it's time to propagate the updates:

```
# instavote
pushd instavote
  git add -Av
  git commit -m "chore: Remove deployment code"
  git push origin HEAD:refs/heads/master
popd

# instavote-deploy
pushd instavote-deploy
  git add -Av
  git commit -am "chore: Import deployment code"
  git push origin HEAD:refs/heads/master
popd

# flux-infra
: This code is not touched as it won't be used anymore
```
