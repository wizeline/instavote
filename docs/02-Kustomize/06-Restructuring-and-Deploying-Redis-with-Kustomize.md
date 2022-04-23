# Restructuring and Deploying Redis with Kustomize

Even if there are no changes to the Redis application it's a good idea to keep the same code structure to take advantage of the Kustomize controller.


```sh
mkdir -pv deploy/redis/{base,dev,staging}
git mv -v deploy/redis/*.yaml deploy/redis/base

# Generate the kustomization file
pushd deploy/redis/base
    kustomize create --autodetect
popd

# Generate the overlay configuration
for env in deploy/redis/{dev,staging}; do
pushd ${env} &>/dev/null
    kustomize create --resources ../base
popd &>/dev/null
done

# Finally commit and publish the updates
git add deploy
git commit -m "refactor: Redis to use Kustomizations"
git push origin HEAD:refs/heads/main
```

## Update the Kustomizations to fetch the new folder structure

Again, now that we have changed the redis codebase we need to update the paths for the Kustomizations:

```sh
cd flux-infra
for env in dev staging; do
    sed -I '' -E "s#path: ./deploy/redis#&/${env}#g" clusters/${env}/redis-${env}-kustomization.yaml
done

# Commit and Promote
git add clusters/
git commit -m "refactor: Update the Redis paths"
git push origin HEAD:refs/heads/main
```

Now just wait for the next reconciliation to happen.
