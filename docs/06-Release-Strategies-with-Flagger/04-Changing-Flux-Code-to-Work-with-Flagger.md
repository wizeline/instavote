# Changing Flux Code to Work with Flagger

There are certain modifications that need to be done in order to be compatible
with Flagger. This is required since Flagger will take control over the
`Service` objects that were previously handled by Flux.

Now for every Deployment we'll have these services enabled for the Vote application:

* vote
* vote-primary
* vote-canary

Also we need to set the `ReplicaCount` to 0 for Flagger to take over the RollOut

And Finally set an Horizontal Pod Autoscaler to take control of the number of
replicas required based on behaviour.


## Update the Vote application

Swithc to the `instavote-deploy` vote application

```sh
cd instavote-deploy/kustomize/vote/base
```

### Generate an Horizontal Pod Autoscaler (HPA)

```sh
cat > hpa.yaml << EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vote
  namespace: instavote
spec:
  maxReplicas: 5
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vote
EOF

# Add the resource to the kustomization
kustomize edit add resource hpa.yaml
```

### Remove the Service configuration

```sh
kustomize edit remove resource service.yaml
```

### Update the Staging environment

Here two thing need to be updated
1. Remove the `service.yaml` reference
1. Set the `replicas.count` to 0

```sh
cd ../staging

# Remove service
sed -i '' -E \
    -e '/service.yaml/d' \
    -e "s/count:.*/count: 0/g" \
    kustomization.yaml
```

### Commit and Publish the changes

```sh
git add -Av
git commit -am "refactor: Enable Flagger Pre-Requisites"
git push origin HEAD:refs/heads/main
```

## Verify the changes were applied

By executing the previous stes we should achieve the next state:

### No services for vote
```sh
$ kubectl get svc -n instavote --selector app=vote
No resources found in instavote namespace.
```
### No pods for vote and replicas to 0

```sh
$ kubectl get pods -n instavote --selector app-vote
No resources found in instavote namespace.

$ kubectl get deploy vote -n instavote
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
vote   0/0     0            0           115m
```

### Horizontal Pod AutoScaler generated
$ kubectl get hpa vote -n instavote
NAME   REFERENCE         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
vote   Deployment/vote   <unknown>/80%   5         10        0          4m31s
