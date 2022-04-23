# Triggering Blue/Green Deployment

Now is all set to test a new deployment, for this you might want to set two
terminals to see the deployment in action

## Terminal 1

This command will show you the components for `vote` as they are analyzed by Flagger Canary (at
the buttom) and deployed (at the top). YOu should expect HPA start deploying new
pods and Flagger to run Analysis on them based on the acceptance and load tests
we previously defined.

```sh
watch 'kubectl get all -o wide \
  -l "kustomize.toolkit.fluxcd.io/name=vote-staging"; \
  kubectl describe canary vote | tail -n 20'
```

## Terminal 2

Here you'll be able to see the Ingress moving the traffict between Deployments
once the RollOut is Completed:

```sh
watch kubectl describe ing
```

## Initiate a Deployment

For this, you just need to change one setting in any part of the Kustomize Overlays
definitions for staging, e.g.: Updating the options in `./kustomize/vote/staging/options.env`

Check for messages like:

```sh

Message:               New revision detected, progressing canary analysis.


Normal   Synced  54s                flagger  New revision detected! Scaling up vote.instavote
Normal   Synced  24s                flagger  Starting canary analysis for vote.instavote
Normal   Synced  24s                flagger  Pre-rollout check acceptance-test passed
Normal   Synced  24s                flagger  Advance vote.instavote canary iteration 1/5
```
