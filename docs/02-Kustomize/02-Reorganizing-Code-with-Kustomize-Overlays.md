# Reorganizing Code with Kustomize Overlays

The changes required to implement Kustomize and Kustomizations happen in the `instavote` application repository:

```sh
cd instavote
```

## Re-structure your application deployment

### Apply changes to the `vote` application

```sh
cd deploy/vote
mkdir base
git mv deployment.yaml service.yaml base/
```

### Generate Kustomization

### Install kustomize

```sh
# Install via Sourcecode
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

# Install via brew
brew install kustomize
```

### Generate a Kustomization
```sh
cd base
kustomize create --autodetect
# A new file kustomization.yaml shall be created
cat kustomization.yaml
```

### Generate an Overlay for the dev environment
```sh
cd ..
mkdir -p dev && cd dev
# Create a new overlay from the base '../base".

kustomize create --resources ../base
## Add the patchesStrategicMerge property
cat >> kustomization.yaml << EOF

patchesStrategicMerge:
- deployment.yaml
EOF

# Generate the patch
cat >> deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vote
  name: vote
spec:
  replicas: 1
  template:
    spec:
      containers:
      - image: schoolofdevops/vote:v5
        name: vote
EOF
```

# Verify the kustomize construction

Kustomize construcs its output based in the structure declared at `kustomize.yaml`. We can render its preview by executing:

```sh
kustomize build
```

# Push your changes to the remote so Flux can reconcile its components

```sh
git add -Av
git commit -am "refactor: Introduce Kustomizations for vote/dev application"
git push origin HEAD:refs/heads/master
```
