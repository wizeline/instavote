# Bootstrapping Flux Fleet with a Sample Tenant

We're going to start by forking a base template for the Flux Fleet. Go ahead and
fork [flux-fleet](https://github.com/wizeline/flux-fleet) to your own workspace
[here](https://github.com/wizeline/flux-fleet/fork).

This repo will be used as the base for the new bootstraped project:

```sh
flux bootstrap github --owner=${GITHUB_USER} \
  --repository=flux-fleet \
  --branch=main \
  --path=./clusters/staging \
  --personal \
  --log-level=debug \
  --network-policy=false

flux check
```


Now you should be able to see the next resources available:

```sh
$ flux get all
NAME                            READY   MESSAGE                                 REVISION        SUSPENDED
gitrepository/flux-system       True    Fetched revision: deploy/a432b3c        deploy/a432b3c  False

NAME                            READY   MESSAGE                                 REVISION        SUSPENDED
kustomization/flux-system       True    Applied revision: deploy/a432b3c        deploy/a432b3c  False
kustomization/projects          True    Applied revision: deploy/a432b3c        deploy/a432b3c  False
```

Here the interesting part to note is the `kustomization/projects` will will enable us to onboard projects at will in the next steps.
