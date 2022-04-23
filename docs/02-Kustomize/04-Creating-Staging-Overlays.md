# Creating Staging Overlays

Verify you are working in the `staging` context

```sh
$ kubectl config current-context
staging
```

A namespace is required before creating a Kustomization.

```sh
kubectl create ns instavote
```

## Create Overlay for Staging

Move to isnstavote repositry and copy the  contents from dev to staging

```sh
cd instavote
cp -r deploy/vote/dev deploy/vote/staging
# Update the replicas to 8
sed -i '' -E "s@replicas:.*@replicas: 8@g" deploy/vote/staging/deployment.yaml
```

Commit the changes and push them to the remote

```sh
git add deploy
git commit -m "feat: Add Staging Overlay for Vote application"
git push origin HEAD:refs/heads/main
```


## Add the customization for the Staging environment

First we need to add the Sources and Kustomizations to the new environment

```
cd flux-infra
cp clusters/dev/redis-dev-kustomization.yaml clusters/staging/redis-staging-kustomization.yaml
cp clusters/dev/vote-dev-kustomization.yaml clusters/staging/vote-staging-kustomization.yaml
cp clusters/dev/instavote-gitrepository.yaml clusters/staging/instavote-gitrepository.yaml

# Replace occurrences from dev -> staging
find clusters/staging/*.yaml -exec sed -I '' 's/dev/staging/g' {} \;

# Commit and publish the changes
git add clusters/staging
git commit -m "feat: Add staging resources"
git push origin HEAD:refs/heads/main
```

### Reconcile the system and see the components deployed

Now that the sorces have been submitted the components will be automatically reconciled

```sh
# Optionally you can trigger the updates
flux reconcile source git flux-system
```

Verify the componets get deployed
```sh
flux get all

# Check the Kubernetes resources
kubectl get all -n instavote
```
