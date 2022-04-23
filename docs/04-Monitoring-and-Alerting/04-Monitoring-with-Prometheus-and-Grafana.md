# Monitoring with Prometheus & Grafana

We can enable a default Grafana and Prometeus monitoring via a Kustomization


# Add the Grafana sources

Add the Git source from Flux

```sh
flux create source git monitoring \
  --interval=30m \
  --url=https://github.com/fluxcd/flux2 \
  --branch=main
```


Generate the Kustomization for both components

```sh
flux create kustomization monitoring \
  --interval=1h \
  --prune=true \
  --source=monitoring \
  --path="./manifests/monitoring" \
  --health-check="Deployment/prometheus.flux-system" \
  --health-check="Deployment/grafana.flux-system"
```

Once the kustomization is ready, you should be able to see the deployment and
corresponding services avaible. By default they are exposed as a `ClusterIP`
and we can do a quick port-forward to show its stats:

```sh
kubectl port-forward service/grafana 3000:3000
```

Visit the next url to see the cluster stats:

    http://127.0.0.1:3000/d/flux-cluster/flux-cluster-stats?orgId=1
