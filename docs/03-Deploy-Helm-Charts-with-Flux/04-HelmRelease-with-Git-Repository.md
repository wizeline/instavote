# HelmRelease with Git Repository

With the freshy generated chart generated we can call a `HelmRelease` via `GitRepository`:

```sh
cd flux-infra
flux create helmrelease result \
  --source=GitRepository/instavote \
  --chart=./deploy/charts/result \
  --target-namespace=instavote \
  --export > clusters/staging/result-staging-helmrelease.yaml
```

Commit and Upload the changes:

```sh
git add clusters/staging
git commit -am "feat: Add Result HelmRelease object"
git push origin HEAD:refs/heads/main
```


Now the application is complete, you can verify it has been deployed via NodePort
or forwarding the service:

```
kubectl port-forward service/instavote-result 8080:80 &

curl localhost:8080
```
