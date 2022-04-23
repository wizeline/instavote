# Clean Up Staging Cluster

In order to port the cluster to the new structute, we need to clean-up the previous
resources in our current cluster. This can be acomplished by using the `flux unistall`
command in one step

```sh
# Run a Dry-Run to see the components which will be uninstalled

$ flux uninstall --namespace=flux-system --dry-run
► deleting components in flux-system namespace
✔ Deployment/flux-system/helm-controller deleted (dry run)
✔ Deployment/flux-system/kustomize-controller deleted (dry run)
✔ Deployment/flux-system/notification-controller deleted (dry run)
✔ Deployment/flux-system/source-controller deleted (dry run)
✔ Service/flux-system/notification-controller deleted (dry run)
✔ Service/flux-system/source-controller deleted (dry run)
✔ Service/flux-system/webhook-receiver deleted (dry run)
✔ ServiceAccount/flux-system/helm-controller deleted (dry run)
✔ ServiceAccount/flux-system/kustomize-controller deleted (dry run)
✔ ServiceAccount/flux-system/notification-controller deleted (dry run)
✔ ServiceAccount/flux-system/source-controller deleted (dry run)
✔ ClusterRole/crd-controller-flux-system deleted (dry run)
✔ ClusterRoleBinding/cluster-reconciler-flux-system deleted (dry run)
✔ ClusterRoleBinding/crd-controller-flux-system deleted (dry run)
► deleting toolkit.fluxcd.io finalizers in all namespaces
✔ GitRepository/flux-system/flux-system finalizers deleted (dry run)
✔ GitRepository/flux-system/instavote finalizers deleted (dry run)
✔ HelmRepository/flux-system/helm-charts finalizers deleted (dry run)
✔ HelmChart/flux-system/flux-system-db finalizers deleted (dry run)
✔ HelmChart/flux-system/flux-system-result finalizers deleted (dry run)
✔ Kustomization/flux-system/flux-system finalizers deleted (dry run)
✔ Kustomization/flux-system/redis-staging finalizers deleted (dry run)
✔ Kustomization/flux-system/vote-staging finalizers deleted (dry run)
✔ Kustomization/flux-system/worker-staging finalizers deleted (dry run)
✔ HelmRelease/flux-system/db finalizers deleted (dry run)
✔ HelmRelease/flux-system/result finalizers deleted (dry run)
► deleting toolkit.fluxcd.io custom resource definitions
✔ CustomResourceDefinition/alerts.notification.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/buckets.source.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/gitrepositories.source.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/helmcharts.source.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/helmreleases.helm.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/helmrepositories.source.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/kustomizations.kustomize.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/providers.notification.toolkit.fluxcd.io deleted (dry run)
✔ CustomResourceDefinition/receivers.notification.toolkit.fluxcd.io deleted (dry run)
✔ Namespace/flux-system deleted (dry run)
✔ uninstall finished


# Now proceed with the Uninstall
flux uninstall --namespace=flux-system

# Do not forget to delete the namespace for instavote
$ kubectl delete ns instavote
namespace "instavote" deleted

```
