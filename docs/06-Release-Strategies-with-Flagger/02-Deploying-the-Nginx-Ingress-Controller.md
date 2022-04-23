# Deploying the Nginx Ingress Controller

We'll follow a similar GitOps approach to install the Ngix Controller in front of our application:


## Generating the Pull-Request

In your forked repository, there must exist a branch called `nginx-ingress` which already contains the configuration to install the components. We can create a Pul-Request directly to see its contintents, go to this URL to generate the Request:

```bash
# Where GITHUB_USER matches your user repository ownership

https://github.com/${GITHUB_USER}/flux-fleet/compare/main...nginx-ingress

```

The next file should be generated:

```sh
infra/base/nginx-ingress
├── kustomization.yaml    # k8s: Kustomization
├── nginx-ingress-helmrelease.yaml  # fluxcd: HelmRelease
└── nginx-ingress-helmrepository.yaml
infra/staging
├── flagger.yaml
├── kustomization.yaml
└── nginx-ingress-helmrelease.yaml

0 directories, 3 files
```

Where:

* `infra/base/nginx-ingress/kustomization.yaml`
  * Adds the kustomization resources such as `HelmRelease` and `HelmRepository`
* `infra/base/nginx-ingress/nginx-ingress-helmrelease.yaml`
  * Defines a `HelmRelese` for the `ingress-nginx` chart
* `infra/base/nginx-ingress/nginx-ingress-helmrepository.yaml`
  * Adds the source for the `ingress-nginx` chart via [ingress-nginx](https://kubernetes.github.io/ingress-nginx)
* `infra/staging/kustomization.yaml`
  * Enables the sources for `nginx-ingress` to be considered
* `infra/staging/nginx-ingress-helmrelease.yaml`
  * Cofigures the `HelmRelease` for the staging environment


## Monitor the Flagger installation
Proceed with the merge and see the resources applied:

### Flux Helm Resources

```sh
$ flux get all -n flagger-system                                                                                                                                                               Manuels-MacBook-Pro.local: Sat May 14 14:20:16 2022

NAME                            READY   MESSAGE                                                                                 REVISION                                                                SUSPENDED
helmrepository/nginx-ingress    True    Fetched revision: 280bcf9f7edb7b98ca3588b66c48f7875324d80674df593ae54127ce78d3eba7      280bcf9f7edb7b98ca3588b66c48f7875324d80674df593ae54127ce78d3eba7        False

NAME                                    READY   MESSAGE                                                 REVISION        SUSPENDED
helmchart/flagger-system-nginx-ingress  True    Pulled 'ingress-nginx' chart with version '4.1.1'.      4.1.1           False

NAME                            READY   MESSAGE                                 REVISION        SUSPENDED
helmrelease/nginx-ingress       True    Release reconciliation succeeded        4.1.1           False
```

### Namespace and Resouces

```sh
$ kubectl get all -n flagger-system
NAME                                                                  READY   STATUS    RESTARTS   AGE
pod/flagger-56b54c646f-shtcp                                          1/1     Running   0          31m
pod/flagger-loadtester-7647f5bd56-m5xbd                               1/1     Running   0          31m
pod/flagger-prometheus-5fb75d78f8-prdf2                               1/1     Running   0          31m
pod/flagger-system-nginx-ingress-ingress-nginx-controller-54789qbff   1/1     Running   0          4m14s

NAME                                                                    TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
service/flagger-loadtester                                              ClusterIP      10.98.253.194   <none>         80/TCP                       31m
service/flagger-prometheus                                              ClusterIP      10.107.35.2     <none>         9090/TCP                     31m
service/flagger-system-nginx-ingress-ingress-nginx-controller           LoadBalancer   10.107.7.146    10.107.7.146   80:31080/TCP,443:31443/TCP   4m15s
service/flagger-system-nginx-ingress-ingress-nginx-controller-metrics   ClusterIP      10.104.72.191   <none>         10254/TCP                    4m15s

NAME                                                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/flagger                                                 1/1     1            1           31m
deployment.apps/flagger-loadtester                                      1/1     1            1           31m
deployment.apps/flagger-prometheus                                      1/1     1            1           31m
deployment.apps/flagger-system-nginx-ingress-ingress-nginx-controller   1/1     1            1           4m15s

NAME                                                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/flagger-56b54c646f                                                 1         1         1       31m
replicaset.apps/flagger-loadtester-7647f5bd56                                      1         1         1       31m
replicaset.apps/flagger-prometheus-5fb75d78f8                                      1         1         1       31m
replicaset.apps/flagger-system-nginx-ingress-ingress-nginx-controller-5478fb857d   1         1         1       4m15s
```

## Importan Note

If you are using `minikube` do not forget to enable the `LoadBalancer` resources to be visible via `minikube tunnel`


### Diable all other `LoadBalancer` resources inthe application

Now that we have an Ingress Controller to expose our application, we'll need to disable the `LoadBalancer` service exposed for the `vote` application:

```sh
cd instavote-deploy
sed -i '' 's/type: LoadBalancer/type: NodePort/' ./kustomize/vote/staging/service.yaml
git add ./kustomize/vote/staging/service.yaml
git commit -am "chore: Convert LoadBalancer to NodePort for Vote service"
git push origin HEAD:refs/heads/main
```
