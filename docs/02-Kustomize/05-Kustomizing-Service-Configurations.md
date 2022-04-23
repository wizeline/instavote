# Kustomizing Service Configurations

## Add an overlay file for the Services in staging

```sh
cd instavote
cat > deploy/vote/staging/service.yaml << EOF
apiVersion: v1
kind: Service
metadata:
  name: vote
spec:
  ports:
  - name: "80"
    nodePort: 30000
    port: 80
    protocol: TCP
    targetPort: 80
  type: LoadBalancer
EOF
```

## Add the patchesStrategicMerge property
```sh
cat >> deploy/vote/staging/kustomization.yaml << EOF
- service.yaml
EOF
```

## Verify  the changes were applied
```sh
kustomize build deploy/vote/staging
```

## Commit and Apply the changes
```sh
git add deploy
git commit -am "feat: Add LoadBalancer service for vote/staging"
git push origin HEAD:refs/heads/main
```

## Verify the configurations are uploaded
```sh
kubectl get service/vote -n instavote
```

## Minikube LoadBalancer

In order to expose the LoadBalancer External ip the next command is required:

```sh
# This will require password to create the process
minikube -p staging tunnel

# Now the EXTERNAL-IP should be populated
```
