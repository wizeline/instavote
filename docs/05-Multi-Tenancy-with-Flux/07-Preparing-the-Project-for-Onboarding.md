# Preparing the Project for Onboarding

Finally is time to onboard the project `instavote` to the fleet


## Generate the RBAC configuration

flux comes with an utility to bootrap tenants (project) wich includes the generation
of Namespace, Roles, RoleBindings and ServiceAccouts. This is whate really defines
the boundaries between projects.

```sh

$ cd flux-fleet
$ mkdir -pv projects/base/instavote
projects/base/instavote

# Generate the RBAC
$ flux create tenant instavote --with-namespace=instavote --export | tee projects/base/instavote/rbac.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    toolkit.fluxcd.io/tenant: instavote
  name: instavote

---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    toolkit.fluxcd.io/tenant: instavote
  name: instavote
  namespace: instavote

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    toolkit.fluxcd.io/tenant: instavote
  name: instavote-reconciler
  namespace: instavote
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: gotk:instavote:reconciler
- kind: ServiceAccount
  name: instavote
  namespace: instavote
```

## Add the Deployment Repository references

With the RBAC in place we can now point to the sources for the flux manifests:

```sh
$ flux create source git instavote-deploy \
  --namespace=instavote \
  --url="https://github.com/${GITHUB_USER}/instavote-deploy.git" \
  --branch main \
  --interval 30s \
  --export | tee ./projects/base/instavote/instavote-deploy-gitrepository.yaml

---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: instavote-deploy
  namespace: instavote
spec:
  interval: 30s
  ref:
    branch: main
  url: https://github.com/${GITHUB_USER}/instavote-deploy.git
```

## Add the Base Kustomization Overlay

```sh
$ flux create kustomization instavote-deploy \
    --namespace=instavote \
    --service-account=instavote \
    --source=GitRepository/instavote-deploy \
    --prune=true \
    --path="./flux" \
    --export | tee ./projects/base/instavote/instavote-deploy-kustomization.yaml

---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: instavote-deploy
  namespace: instavote
spec:
  interval: 1m0s
  path: ./flux
  prune: true
  serviceAccountName: instavote
  sourceRef:
    kind: GitRepository
    name: instavote-deploy
```

Generate the kustomization file to stash all the changes:

```sh
pushd projects/base/instavote
  kustomize create --autodetect
popd
```

Verify its outputs:

```sh
$ cat projects/base/instavote/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- instavote-deploy-gitrepository.yaml
- instavote-deploy-kustomization.yaml
- rbac.yaml
```

## Add the Staging Kustomization Overlay

Generate the Kustomization Patch to point to the new path

```sh
cat << EOF | tee ./projects/staging/instavote-deploy-kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: instavote-deploy
  namespace: instavote
spec:
  path: ./flux/staging
EOF
```

Let the Kustomize controller the new settings need to be applied:

```sh
# Add the new resource
sed -i '' 's@resources:@&\n- ../base/instavote@g' projects/staging/kustomization.yaml

# Add the new patchesStrategicMerge
sed -i '' 's@patchesStrategicMerge:@&\n- instavote-deploy-kustomization.yaml@g' projects/staging/kustomization.yaml
```

## Add the missing secret reference

Now that we have migrated the deploy repository to `instavote-deploy` we need to
re-generate the secret for `github-token`. Follow the steps defined at [Auto Updating Git Commit Status](docs/04-Monitoring-and-Alerting/03-Auto-Updating-the-Git-Commit-Status.md) to get the
new token and execute:

```sh
kubectl create secret -n instavote generic github-token --from-literal=token=${NEW_TOKEN}
```

## Commit and Publish your changes

```
git add -Av
git commit -am "chore: Onboard instavote tenant"
git push origin HEAD:refs/heads/main
```
