# Adding Health Checks
-------


# Break the Deployment
Go back to the Application Repository `instavote`
```sh
# Change the tag to a random string
sed -i '' -E "s@vote:.*@vote:$(md5 -q -s $RANDOM)@g" deploy/vote/deployment.yaml
git add deploy/vote/deployment.yaml
git commit -m "build\!: breaking the deployment"
git push origin HEAD:refs/heads/main
```

Reconcile the sources
```sh
flux reconcile kustomization vote-dev
```

Verify the Kustomizations
```sh
flux get kustomizations
```

Altought the Kustomization reports a `Ready:True` the deployment returns a failed state `ImagePullBackOff`

```sh
kubectl get pod,deploy
kubectl rollout status deploy vote -n kube-system
```

Now is time to set a HealthCheck, go back to your `flux-infra` repository

```sh
# Display the changes to be applied
flux create kustomization vote-dev \
    --source=instavote \
    --path="./deploy/vote" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/vote.instavote"
    --export

# Update the Configuration File
flux create kustomization vote-dev \
    --source=instavote \
    --path="./deploy/vote" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/vote.instavote" \
    --export > clusters/dev/vote-dev-kustomization.yaml

# See the differences
git diff clusters/dev/vote-dev-kustomization.yaml

# Commit the differences to the Cluster
git add clusters/dev/vote-dev-kustomization.yaml
git commit -am "feat: Add healtchecks to the Kustomization"
git push origin HEAD:refs/heads/main
```
------
With the changes in place we can reconcile and check the differences in `Ready:Unknown`

```sh
flux reconcile kustomization flux-system

# Not the READY
flux get kustomizations
```

-----
## Revert the broken deployment
Now we have to revert the breaking change in `instavote`

```sh
git revert HEAD
git push origin HEAD:refs/heads/main
```
