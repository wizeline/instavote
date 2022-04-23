# Slack Notifications


In order to set-up a Slack notification we need to perfm 3 steps:

1. Generate an Incomming Webhook
2. Provider Notification (type slack)
3. Alerts

Be aware that you need admin access to the channel in order to establish the app
and the Webhook configuration.

## Generate an Incomming Webhook

Follow the instructions to generate a token what will be used as a secret.

    https://api.slack.com/messaging/webhooks#create_a_webhook

With the value at hand, generate a secret in the cluster via:

```sh
kubectl create secret generic slack-url --from-literal=adress=https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX -n flux-system
```

Make sure to replace the value for the hook with your ouw settings.


## Define a Notification Provider

This CRD will consume the secret to access to communicate with Slack. First verify that you have the notification controller

```sh

$ flux check --components notification-controller

► checking prerequisites
✗ flux 0.27.3 <0.29.3 (new version is available, please upgrade)
✔ Kubernetes 1.23.3 >=1.20.6-0
► checking controllers
✔ helm-controller: deployment ready
► ghcr.io/fluxcd/helm-controller:v0.17.1
✔ kustomize-controller: deployment ready
► ghcr.io/fluxcd/kustomize-controller:v0.21.1
✔ notification-controller: deployment ready
► ghcr.io/fluxcd/notification-controller:v0.22.2
✔ source-controller: deployment ready
► ghcr.io/fluxcd/source-controller:v0.21.2
✔ all checks passed
```

Whith the values verified we can proceed to generate the Notification Provider:

```sh
flux create alert-provider slack \
    --type=slack \
    --secret-ref=slack-url \
    --channel=${your_channel_name} \
    --export
```

Verify the output:

```yaml
---
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  channel: fluxcd
  secretRef:
    name: slack-url
  type: slack
```

If correct proceed to save the configuration and upload it to the `flux-infra` repository

```sh

flux create alert-provider slack \
    --type=slack \
    --secret-ref=slack-url \
    --channel=${your_channel_name} \
    --export > clusters/staging/slack-staging-alert-provider.yaml

# Commit and publish
git add clusters/staging
git commit -am "feat: Add Alert Provider for Slack"
git push origin HEAD:refs/heads/main
```

Verify the Provider is installed correectly

```sh
flux get alert-providers
```

Note that the provider does not have any impact on the workflows as it only acts
as the exit gate to communicate, we still need to complete the configuration to
send messages.

# Create an Alert

This component links the Sources against the Notification Provider to be activated
in for some triggers. Lets create an Alert for all `info` events from all of our
sources:

```sh
flux create alert slack-notification \
    --event-source "Kustomization/*" \
    --event-source "GitRepository/*" \
    --event-source "HelmRelease/*" \
    --provider-ref=slack \
    --event-severity=info \
    --export
```

Verify its outputs:

```yaml
---
apiVersion: notification.toolkit.fluxcd.io/v1beta1
kind: Alert
metadata:
  name: slack-notification
  namespace: flux-system
spec:
  eventSeverity: info
  eventSources:
  - kind: Kustomization
    name: '*'
  - kind: GitRepository
    name: '*'
  - kind: HelmRelease
    name: '*'
  providerRef:
    name: slack
```

Generate the source file, and make it available for reconciliation

```sh
flux create alert slack-notification \
    --event-source "Kustomization/*" \
    --event-source "GitRepository/*" \
    --event-source "HelmRelease/*" \
    --provider-ref=slack \
    --event-severity=info \
    --export > clusters/staging/slack-notification-staging-alert.yaml

# Commit and publish
git add clusters/staging
git commit -am "feat: Add Alert for Slack"
git push origin HEAD:refs/heads/main
```

Finally verify the Alert is being generated

```sh
flux get alerts
```
