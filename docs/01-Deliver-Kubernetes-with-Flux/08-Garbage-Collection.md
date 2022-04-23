# Garbage Collection

Flux automatically deletes objects which are no longer part of the Kustomizations.
Now we can try deleting the Service Object and see how it disappears from the cluster.

Go to the `instavote` repository
```sh
git rm deploy/vote/service.yaml
git commit -am "chore: delete service object"
git push origin HEAD:refs/heads/main
```

In less than a minute you shall see how the Service disappears.

```sh
kubectl get svs
```

Now recover the service by restoring the file and committing the changes
```sh
git revert HEAD --no-edit
git push origin HEAD:refs/heads/main
```
