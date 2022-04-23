# Initializing the Instavote Deploy Repository

For this exercise we'll start by onboarding the instavode deployment code in a
new repository, click here to fork it: [instavote-deploy](https://github.com/wizeline/instavote-deploy/fork)



With the repository clone proceed to exectue the next scrip to create the scafold
to initialize the instavote project:

```sh
cat > flux/base/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Deployment Manifests which are Synced by Flux to Deploy to a Kubernetes Environment.

Example of the resources you could add here,

Sources:
  - GitRepository
  - HelmRepository
  - Bucket

Deployment:
  - HelmRelease
  - Kustomization (Flux App Deployment)

Notification:
  - Alert
  - Provider
  - Receiver

Image Automation:
  - ImageRepository
  - ImagePolicy
  - ImageUpdateAutomation

EOF

cat > flux/staging/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Kustomize Overlays for staging environemnt to flux sync  manifests defined in ../base.
If you add a patch file, ensure you update kustomization.yaml accordingly.
EOF

cat > flux/production/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Kustomize Overlays for staging environemnt to  manifests flux sync  defined in ../base.
If you add a patch file, ensure you update kustomization.yaml accordingly.
EOF


cat > flux/staging/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../base
EOF


cat > flux/production/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../base
EOF



cat > flux/base/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Kubernetes Manifests YAML.

Examples of resources that you would add here,
  - Deployment
  - Service
  - StatefulSet
  - DaemonSet
  - Job
  - CronJob
  - PersistentVolumeClaim

In fact it could be any resource that you would apply to kubernetes.

EOF

cat > flux/staging/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Kustomize Overlays for staging environemnt to  manifests defined in ../base.
If you add a patch file, ensure you update kustomization.yaml accordingly.
EOF

cat > flux/production/README.md <<EOF
## Kubernetes Deployment Manifests
This path is to add Kustomize Overlays for staging environemnt to  manifests defined in ../base.
If you add a patch file, ensure you update kustomization.yaml accordingly.
EOF

cat > flux/staging/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../base
EOF


cat > flux/production/kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../base
EOF

cat > helm/charts/README.md <<EOF
## Helm Charts
Add your helm charts here.
EOF
```

These files shall be updated:

- flux/base/README.md
- flux/production/README.md
- flux/production/kustomization.yaml
- flux/staging/README.md
- flux/staging/kustomization.yaml
- helm/charts/README.md


And we're ready to commit and promote the changes:

```sh
git add -Av
git commit -m "chore: Add onboarding flux infrastructure"
git push origin HEAD:refs/heads/main
```
