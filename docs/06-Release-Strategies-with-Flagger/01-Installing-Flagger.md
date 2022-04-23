# Installing Flagger

Although there are various options defined at [flagger-install-on-kubernetes](https://docs.flagger.app/install/flagger-install-on-kubernetes) we're going to follow the _GitOps_ approach by generating a Pull-Request defining the components needed to be installed via `Kustomization`:


## Generating the Pull-Request

In your forked repository, there must exist a branch called `flagger` which already contains the configuration to install the components. We can create a Pul-Request directly to see its contintents, go to this URL to generate the Request:

```bash
# Where GITHUB_USER matches your user repository ownership

https://github.com/${GITHUB_USER}/flux-fleet/compare/main...flagger

```

The next file should be generated:

```sh
clusters/staging
├── infra.yaml                                  # fluxcd: Kustomization
infra
├── base
│   ├── flagger
│   │   ├── kustomization.yaml                  # k8s: Kustomization
│   │   └── namespace.yaml                      # k8s: Namespace
└── staging
    ├── flagger.yaml                            # k8s: Deployment
    └── kustomization.yaml                      # k8s: Kustomization

4 directories, 5 files
```

Where:

* `clusters/infra.yaml`
  * Defines a Kustomization in the flux-system namespace
* `infra/base/flagger/kustomization.yaml`
  * Includes the resources to add `tester`, `prometheus`, `flagger`, and its namespace to reside
* `infra/base/flagger/namespace.yaml`
  * Enables and disables annotations and labels for the K8S resources managed by the service mesh.
* `infra/staging/flagger.yaml`
  * Configures the `Deployment` to our needs in settings such as the Prometheus server and log-level
* `infra/staging/kustomization.yaml`
  * Links the resources and applies the `patchesStrategicMerge`

## Monitor the Flagger installation
Proceed with the merge and see the resources applied:

### Flux Kustomization

```sh
$ flux get kustomizations infra
NAME    READY   MESSAGE                                 REVISION        SUSPENDED
infra   True    Applied revision: deploy/ca5a2bc        deploy/ca5a2bc  False
```

### Namespace and Resouces

```sh
$ kubectl get ns flagger-system
NAME             STATUS   AGE
flagger-system   Active   5m24s

$ kubectl get all -n flagger-system
NAME                                      READY   STATUS    RESTARTS   AGE
pod/flagger-56b54c646f-qh475              1/1     Running   0          6m13s
pod/flagger-loadtester-7647f5bd56-bnfrw   1/1     Running   0          6m13s
pod/flagger-prometheus-5fb75d78f8-jx6j4   1/1     Running   0          6m13s

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/flagger-loadtester   ClusterIP   10.105.233.89    <none>        80/TCP     6m13s
service/flagger-prometheus   ClusterIP   10.102.133.157   <none>        9090/TCP   6m13s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/flagger              1/1     1            1           6m13s
deployment.apps/flagger-loadtester   1/1     1            1           6m13s
deployment.apps/flagger-prometheus   1/1     1            1           6m13s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/flagger-56b54c646f              1         1         1       6m13s
replicaset.apps/flagger-loadtester-7647f5bd56   1         1         1       6m13s
replicaset.apps/flagger-prometheus-5fb75d78f8   1         1         1       6m13s
```
