# Setting Common Labels and Annotations

We're also capable of adding meta information to the objects

```sh
cat >> deploy/vote/staging/kustomization.yaml << EOF

commonAnnotations:
  supported-by: support@fluxcd.io
EOF
```

Verify the changes are correct by showing the output via:

```sh
kustomize build deploy/vote/staging/staging/
```

## Commit and deliver the changes
```sh
git add deploy/
git commit -m "chore: Add Annotations to vote application in the staging environment"
git push origin HEAD:refs/heads/main
```

## Label updates

Beaware that labels applied to a Deployments are immutable so any changes to them would require a new Deployment to be generated. In our case we can take advantage of Flux capabilities by deleting the deployment and having the system creating a new one right after.

```sh
cat >> deploy/vote/staging/kustomization.yaml << EOF
commonLabels:
  project: instavote
  env: staging
EOF
```
