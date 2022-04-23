# Images and Namespaces

Now we're going to update the image used by redis by updating the deployment information. There are several options to update like containers, versions, tags. There's also the option to stamp the resources with a new namespace.

In our case we're going to change the base image for redis and declare the namespace directly in the resource:

```sh
cat >> deploy/redis/staging/kustomization.yaml << EOF

images:
  - name: redis:alpine
    newTag: bullseye

namespace: instavote
EOF
```

Verify the changes are correct by showing the output via:

```sh
kustomize build deploy/redis/staging/
```

## Commit and deliver the changes
```sh
git add deploy/
git commit -m "chore: Update redis image to bullseye in staging"
git push origin HEAD:refs/heads/main
```
