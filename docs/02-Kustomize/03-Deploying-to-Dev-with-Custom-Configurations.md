# Deploying to Dev with Custom Configurations

Now that we have updated the values for `deploy/vote` the Kustomize component should be broken:

```plain
flux get kustomizations --context minikube
NAME            READY   MESSAGE                                                 REVISION                                        SUSPENDED
flux-system     True    Applied revision: main/c913cb7                          main/c913cb7                                    False
redis-dev       True    Applied revision: main/b7b3340                          main/b7b3340                                    False
vote-dev        False   kustomize build failed: accumulating resources: accumulation err='accumulating resources from './dev': read /tmp/vote-dev2413154815/deploy/vote/dev: is a directory': recursed merging from path '/tmp/vote-dev2413154815/deploy/vote/dev': may not add resource with an already registered id: Deployment.v1.apps/vote.[noNs]      main/616357b9d4d08500e2b7e1bf10a57fae99f6db7a       False
```

If we go in detail we'll see the file structure is broken:

```sh
flux get kustomizations vote-dev  --context minikube
NAME    	READY	MESSAGE REVISION SUSPENDED

vote-dev        False   kustomize build failed: accumulating resources: accumulation err='accumulating resources from './dev': read /tmp/vote-dev2413154815/deploy/vote/dev: is a directory': recursed merging from path '/tmp/vote-dev2413154815/deploy/vote/dev': may not add resource with an already registered id: Deployment.v1.apps/vote.[noNs]  main/616357b9d4d08500e2b7e1bf10a57fae99f6db7a   False
```

## Fix the Kustomization

Move to the cluster configuration repository
```sh
cd flux-infra
# Refresh the contents
git pull
# Update the Kustomization by setting the new path
flux create kustomization vote-dev \
    --source=instavote \
    --path="./deploy/vote/dev" \
    --prune=true \
    --interval=1m \
    --target-namespace=instavote \
    --health-check="Deployment/vote.instavote" \
    --depends-on=redis-dev \
    --export > clusters/dev/vote-dev-kustomization.yaml
```

Now verify the differnces via `git diff`
```diff
diff --git a/clusters/dev/vote-dev-kustomization.yaml b/clusters/dev/vote-dev-kustomization.yaml
index 809acb6..158e4e7 100644
--- a/clusters/dev/vote-dev-kustomization.yaml
+++ b/clusters/dev/vote-dev-kustomization.yaml
@@ -12,10 +12,11 @@ spec:
     name: vote
     namespace: instavote
   interval: 1m0s
-  path: ./deploy/vote
+  path: ./deploy/vote/dev
   prune: true
   sourceRef:
     kind: GitRepository
     name: instavote
   targetNamespace: instavote
   timeout: 2m0s
```

### Commit and Deploy the changes

```sh
git commit -am "fix: Update paths for Vote Kustomization"
git push origin HEAD:refs/heads/master
```


### Verify the Kustomization gets applied

```sh
# Reconcile the sources and Kustomizations
flux reconcile source git flux-system
flux reconcile kustomization vote-dev --context minikube
```

Now the Kustomization should be back to normal along with the updated resources

```sh
flux get kustomizations --context minikube
NAME            READY   MESSAGE                         REVISION        SUSPENDED
flux-system     True    Applied revision: main/decc368  main/decc368    False
redis-dev       True    Applied revision: main/6a382c7  main/6a382c7    False
vote-dev        True    Applied revision: main/6a382c7  main/6a382c7    False
```

### Verify the updates were registered in the app

```sh
kubectl get pods --selector app=vote -n instavote --context minikube -o jsonpath="{.items[*].spec.containers[*].image}"
```

Expose the port and run a local check

```sh
$ port-forward svc/vote 8080:80  --context minikube

$ curl localhost:8080 -s | grep  Version
        <h2> App Version v5</h2>
```
