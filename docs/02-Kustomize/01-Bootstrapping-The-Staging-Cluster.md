# Bootstrapping the Staging Cluster

## Spin a new cluster to emulate a new environment

### Generate a new cluster for staging
```sh
minikube start --nodes 2 -p staging
```

### Switch your context with kubectx
```sh
# List your contexts
kubectl config get-contexts

# Swith to the new generated context
kubectx staging

# In case you don't have kubectx use the native command
kubectl config use-context staging

# Verify the context is available
kubectl config current-context
```

## Bootstrap Flux with a new Cluster

```sh
flux bootstrap github --owner=${GITHUB_USER} \
  --repository=flux-infra \
  --branch=main \
  --path=./clusters/staging \
  --personal \
  --log-level=debug \
  --network-policy=false
  # Notice how the components are not declared
```

The controllers should be visible at

```sh
kubectl get all -n flux-system
```

Verify the installation
```sh
flux check
```

You can boostrap as many times as you want, as the process is idempotent.


Additionally new components are now installed:

```sh
kubectl get crds
```

Now components for `helmreleases`, `helmrepositories`, `helmchart` should be visible.
