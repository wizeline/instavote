# Auto-Updating the Git Commit Status

First of all we need to generate a new Auth Token with `repo` cababilities wich
will be set as a secret in kubernetes

## Generate an auth token

Check all the `repo` attributes in https://github.com/settings/tokens/new

Save the result token as a secret in Kubernetes

```sh
kubectl create secret -n flux-system generic github-token --from-literal=token=${NEW_TOKEN}
```

## Add an Alert Provider

```sh
flux create alert-provider github-instavote \
    --type github \
    --address="https://github.com/${GITHUB_USER}/instavote.git" \
    --secret-ref=github-token \
    --export | tee clusters/staging/github-instavote-staging-alert-provider.yaml
```

Verify the outputs and providers:

```yaml
---
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Provider
metadata:
  name: github-instavote
  namespace: flux-system
spec:
  address: https://github.com/${GITHUB_USER}/instavote.git
  secretRef:
    name: github-token
  type: github
```

Commit and propagate the changes

```sh
git add clusters/staging
git commit -m "feat: Add github-instavote Alert Provider"
git push origin HEAD:refs/heads/main
```

Wait for the Reconciliation to take place and verify the alert-provider is visible:

```sh
flux get alert-providers
```

# Add Alerts

Now that we have our gate to GitHub we can start adding Alerts to the commit
messages. Keep in mind that only the Kustomizations will be able to send
information as they are the only ones containing a SHA:

```sh
cd flux-infra

for kustomize in redis vote worker; do
	flux create alert ${kustomize}-staging \
	  --provider-ref=github-instavote \
	  --event-severity=info \
	  --event-source "Kustomization/${kustomize}-staging" \
	  --export | tee clusters/staging/${kustomize}-staging-alert.yaml
 done
```

Commit, Publish and Reconcile the Changes

```sh
git add clusters/staging
git commit -m "feat: Add Alerts for Kustomizations"
git push origin HEAD:refs/heads/main

# Reconcile sources and check for alerts
flux reconcile source git flux-system
flux get alerts
```

## Verify the Alerts

Have fun and change the setting for any part of the vote, redis or worker application
and you'll see the magic in place ;)
