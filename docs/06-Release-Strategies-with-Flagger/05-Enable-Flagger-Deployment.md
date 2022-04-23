# Enable Flagger Deployment

Now is time to connect all the dots. Go to the `instavote-deploy` repository and
apply the next set of changes

``sh
cd instavote-deploy
``

## Generate teh vote-canary spec

```sh
cd kustomize/vote/base
cat > vote-canary.yaml << EOF
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: vote
  namespace: instavote
spec:
  provider: nginx
  # deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vote
  # ingress reference
  ingressRef:
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    name: vote
  # HPA reference (optional)
  autoscalerRef:
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    name: vote
  # the maximum time in seconds for the canary deployment
  # to make progress before it is rollback (default 600s)
  progressDeadlineSeconds: 60
  service:
    # ClusterIP port number
    port: 80
    # container port number or name
    targetPort: 80
  analysis:
    # schedule interval (default 60s)
    interval: 30s
    # max number of failed metric checks before rollback
    threshold: 3
    # --- enable this for blue/green
    # number of checks to run before rollback
    #iterations: 5
    # ---
    # === enable this for canary
    # max traffic percentage routed to canary
    # percentage (0-100)
    #maxWeight: 50
    # canary increment step
    # percentage (0-100)
    #stepWeight: 5
    # ===
    metrics:
    - name: request-success-rate
      # minimum req success rate (non 5xx responses)
      # percentage (0-100)
      thresholdRange:
        min: 99
      interval: 1m
    # testing (optional)
    webhooks:
      - name: acceptance-test
        type: pre-rollout
        url: http://flagger-loadtester.flagger-system/
        timeout: 30s
        metadata:
          type: bash
          cmd: "curl http://vote-canary.instavote/ | grep choice"
      - name: load-test
        url: http://flagger-loadtester.flagger-system/
        timeout: 5s
        metadata:
          cmd: "hey -z 30s -q 10 -c 2 http://vote-canary.instavote/"
EOF

# Do not forget to add the Resource
kustomize edit add resource vote-canary.yaml
```

## Apply the Overlay configurations for Staging

```sh
cd ../staging
cat > vote-canary.yaml << EOF
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: vote
  namespace: instavote
spec:
  analysis:
    interval: 30s
    threshold: 3
    iterations: 5
EOF

# Do not forget to update the patchesStrategicMerge
sed -i '' 's@patchesStrategicMerge:@&\n- vote-canary.yaml@g' kustomization.yaml
```

## Commit and Publish your changes


```sh
git add -Av
git commit -am "build: Enable Flagger Deployments"
git push origin HEAD:refs/heads/maint"
```


## Verify the Deployment is Active

We'll check for all of the components being available


### Check for the Ingress

Now there's another ingres enabled for primary

```sh
$ kubectl get ingress -n instavote
NAME          CLASS    HOSTS              ADDRESS         PORTS   AGE
vote          <none>   vote.example.com   10.103.190.27   80      69m
vote-canary   <none>   vote.example.com   10.103.190.27   80      11m
```

See the details, on how the can `vote-canary` points nowhere for now, while `vote`
points to the running version for the application.

```sh
$ kubectl describe ingress
Name:             vote
Labels:           app=vote
                  kustomize.toolkit.fluxcd.io/name=vote-staging
                  kustomize.toolkit.fluxcd.io/namespace=instavote
Namespace:        instavote
Address:          10.107.7.146
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /   vote:80 (172.17.0.11:80,172.17.0.12:80,172.17.0.13:80 + 2 more...)
Annotations:  kubernetes.io/ingress.class: nginx
              supported-by: support@fluxcd.io
Events:       <none>


Name:             vote-canary
Labels:           app=vote
                  kustomize.toolkit.fluxcd.io/name=vote-staging
                  kustomize.toolkit.fluxcd.io/namespace=instavote
Namespace:        instavote
Address:          10.107.7.146
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /   vote-canary:80 (<none>)
Annotations:  kubernetes.io/ingress.class: nginx
              kustomize.toolkit.fluxcd.io/reconcile: disabled
              nginx.ingress.kubernetes.io/canary: true
              nginx.ingress.kubernetes.io/canary-weight: 0
              supported-by: support@fluxcd.io
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    11m (x2 over 12m)  nginx-ingress-controller  Scheduled for sync
```

### Deployments and Pods

Deployment up and running with HPA attached to it

```sh
$ kubectl describe  deploy/vote-primary -n instavote
Name:                   vote-primary
Namespace:              instavote
CreationTimestamp:      Sat, 14 May 2022 16:07:17 -0500
Labels:                 app=vote-primary
Annotations:            deployment.kubernetes.io/revision: 1
                        supported-by: support@fluxcd.io
Selector:               app=vote-primary
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:       app=vote-primary
  Annotations:  flagger-id: 6e95e822-209f-4100-9c8e-3d88d5a96828
                supported-by: support@fluxcd.io
  Containers:
   vote:
    Image:      schoolofdevops/vote:v5
    Port:       <none>
    Host Port:  <none>
    Environment Variables from:
      vote-options-gk765h5m97-primary  ConfigMap  Optional: true
    Environment:                       <none>
    Mounts:                            <none>
  Volumes:                             <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   vote-primary-547457859 (5/5 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m38s  deployment-controller  Scaled up replica set vote-primary-547457859 to 1
  Normal  ScalingReplicaSet  5m53s  deployment-controller  Scaled up replica set vote-primary-547457859 to 5
```

### Services

Services are generated for the application `primary` and `canary`. Each one
pointing to the corresponding deployment.

```sh
$ kubectl get service -n instavote
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
# db                 ClusterIP   10.98.214.35     <none>        5432/TCP       146m
# instavote-result   NodePort    10.101.211.72    <none>        80:32023/TCP   146m
# redis              ClusterIP   10.96.11.248     <none>        6379/TCP       146m
vote               ClusterIP   10.103.190.205   <none>        80/TCP         5m1s
vote-canary        ClusterIP   10.103.166.13    <none>        80/TCP         5m31s
vote-primary       ClusterIP   10.97.35.29      <none>        80/TCP         5m31s
```

### Horizontal Pod AutoScalers

Now the HPA is fullil via `vote-primary`, and `vote` acts as the backout environment.

```sh
$ kubectl get hpa -n instavote
NAME           REFERENCE                 TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
vote           Deployment/vote           <unknown>/80%   5         10        0          37m
vote-primary   Deployment/vote-primary   <unknown>/80%   5         10        5          7m52s
```
