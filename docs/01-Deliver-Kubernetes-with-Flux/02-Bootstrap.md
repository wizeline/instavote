# Bootstrap the cluster

Check for the pre-requisites:

```sh
flux check --pre
```

Run the next command to boostrap a cluster.

 * _make sure you are running in the right context_

 ```sh
 flux bootstrap github --owner=${GITHUB_USER} \
   --repository=flux-infra \
   --branch=main \
   --path=./clusters/dev \
   --personal \
   --log-level=debug \
   --network-policy=false \
   --components=source-controller,kustomize-controller
```


The controllers should be visible at

```sh
kubectl get all -n flux-system
```

Verify the installation
```sh
flux check
```

You can boostrap as many times as you want, as the process is idempotent.
