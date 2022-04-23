# Updating Scale by Setting Replica Counts

One of the features of Kustomize is to update properties in an easy way. Lets update the `replicas` properties directly from the `kustomization.yaml` instead of patching:


```sh
cat >> deploy/vote/staging/kustomization.yaml << EOF

replicas:
  - name: vote
    count: 10
EOF
```

Verify the changes are correct by showing the output via:

```sh
kustomize build deploy/vote/staging/
```

## Commit and deliver the changes
```sh
git add deploy/
git commit -m "chore: scale up vote replicas to 10 in staging"
git push origin HEAD:refs/heads/main
```
