# Kustomize Controller


Deploy the `vote` application:

```sh
flux create kustomization vote-dev \
    --source=instavote \
    --path="./deploy/vote" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote
```

Verify the Sources and Kustomizations

```sh
flux get sources
flux get kustomizations
flux reconcile kustomization vote-dev
flux reconcile source git instavote
```

Validate the app has been deployed

```sh
kubectl get service/vote  -o wide
```

_if running on Minikube._
```sh
minikube service --all -n instavote
```

Trigger an update to test the CD flow by scaling the replicas to 10:

```sh
sed  -E 's@(replicas:).*@\1 10@g' deploy/vote/deployment.yaml
git add deploy/vote/deployment.yaml
git commit -m "chore: Update vote replicas to 10"
git push origin HEAD:refs/heads/main

# Reconcile
flux reconcile kustomization vote-dev
kubectl get pods -n instavote
```
