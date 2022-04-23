Example Voting (Instavote) App
=========

Architecture
-----

![Architecture diagram](architecture.png)

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Description
-----
![MicroServices](https://user-images.githubusercontent.com/6550812/158926965-b10d798b-7745-4a45-82d2-28d511c3c025.png)

Note
----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.


Tools
-----

* [kubectx/kubens](https://github.com/ahmetb/kubectx)

Instal via hombebrew
```sh
brew install kubectx
```


Contents
-----
- [Deliver Kubernetes with Flux](docs/01-Deliver-Kubernetes-with-Flux)
  - [Install Flux CLI](docs/01-Deliver-Kubernetes-with-Flux/01-Install-Flux-CLI.md)
  - [Bootstrap](docs/01-Deliver-Kubernetes-with-Flux/02-Bootstrap.md)
  - [Verify Controllers](docs/01-Deliver-Kubernetes-with-Flux/03-Verify-Controllers.md)
  - [Create Source Controller](docs/01-Deliver-Kubernetes-with-Flux/04-Create-Source-Controller.md)
  - [Kustomize Controller](docs/01-Deliver-Kubernetes-with-Flux/05-Kustomize-Controller.md)
  - [Exporting Sync Manifests](docs/01-Deliver-Kubernetes-with-Flux/06-Exporting-Sync-Manifests.md)
  - [Adding Health Checks](docs/01-Deliver-Kubernetes-with-Flux/07-Adding-Health-Checks.md)
  - [Garbage Collection](docs/01-Deliver-Kubernetes-with-Flux/08-Garbage-Collection.md)
  - [Defining Sequencing with DependsOn](docs/01-Deliver-Kubernetes-with-Flux/09-Defining-Sequencing-with-DependsOn.md)
  - [Verifying The Application](docs/01-Deliver-Kubernetes-with-Flux/10-Verifying-The-Application.md)
- [Kustomize](docs/02-Kustomize)
  - [Bootstrapping The Staging Cluster](docs/02-Kustomize/01-Bootstrapping-The-Staging-Cluster.md)
  - [Reorganizing Code with Kustomize Overlays](docs/02-Kustomize/02-Reorganizing-Code-with-Kustomize-Overlays.md)
  - [Deploying to Dev with Custom Configurations](docs/02-Kustomize/03-Deploying-to-Dev-with-Custom-Configurations.md)
  - [Creating Staging Overlays](docs/02-Kustomize/04-Creating-Staging-Overlays.md)
  - [Kustomizing Service Configurations](docs/02-Kustomize/05-Kustomizing-Service-Configurations.md)
  - [Restructuring and Deploying Redis with Kustomize](docs/02-Kustomize/06-Restructuring-and-Deploying-Redis-with-Kustomize.md)
  - [Updating Scale by Setting Replica Counts](docs/02-Kustomize/07-Updating-Scale-by-Setting-Replica-Counts.md)
  - [Images and Namespaces](docs/02-Kustomize/08-Images-and-Namespaces.md)
  - [Setting Common Labels and Annotations](docs/02-Kustomize/09-Setting-Common-Labels-and-Annotations.md)
  - [Generating ConfigMaps](docs/02-Kustomize/10-Generating-ConfigMaps.md)
  - [Automating Rollouts on Configuration Changes](docs/02-Kustomize/11-Automating-Rollouts-on-Configuration-Changes.md)
  - [Generate Worker Deployment](docs/02-Kustomize/12-Generate-Worker-Deployment.md)
- [Deploy Helm Charts with Flux](docs/03-Deploy-Helm-Charts-with-Flux)
  - [Defining a Helm Repository as a Source](docs/03-Deploy-Helm-Charts-with-Flux/01-Defining-a-Helm-Repository-as-a-Source.md)
  - [Deploying with HelmRelease](docs/03-Deploy-Helm-Charts-with-Flux/02-Deploying-with-HelmRelease.md)
  - [Deploying Vote Result with ChartSource](docs/03-Deploy-Helm-Charts-with-Flux/03-Deploying-Vote-Result-with-ChartSource.md)
  - [HelmRelease with Git Repository](docs/03-Deploy-Helm-Charts-with-Flux/04-HelmRelease-with-Git-Repository.md)
  - [Features of a Notification Controller](docs/04-Monitoring-and-Alerting/01-Features-of-a-Notification-Controller.md)
- [Monitoring and Alerting](docs/04-Monitoring-and-Alerting)
  - [Features of a Notification Controller](docs/04-Monitoring-and-Alerting/01-Features-of-a-Notification-Controller.md)
  - [Slack Notifications](docs/04-Monitoring-and-Alerting/02-Slack-Notifications.md)
  - [Auto Updating Git Commit Status](docs/04-Monitoring-and-Alerting/03-Auto-Updating-the-Git-Commit-Status.md)
  - [Monitoring with Prometheus & Grafana](docs/04-Monitoring-and-Alerting/04-Monitoring-with-Prometheus-and-Grafana.md)
- [Multi-Tenancy with Flux](docs/05-Multi-Tenancy-with-Flux/)
  - [The Need for Multi Tenancy](docs/05-Multi-Tenancy-with-Flux/01-The-Need-for-Multi-Tenancy.md)
  - [Exporting Sync Manifests](docs/05-Multi-Tenancy-with-Flux/02-Exporting-Sync-Manifests.md)
  - [Clean Up Staging Cluster](docs/05-Multi-Tenancy-with-Flux/03-Clean-Up-Staging-Cluster.md)
  - [Bootstrapping Flux Fleet](docs/05-Multi-Tenancy-with-Flux/04-Bootstrapping-Flux-Fleet.md)
  - [Initializing the Instavote Deploy Repository](docs/05-Multi-Tenancy-with-Flux/05-Initializing-the-Instavote-Deploy-Repository.md)
  - [Migrating the Deployment Code](docs/05-Multi-Tenancy-with-Flux/06-Migrating-the-Deployment-Code.md)
  - [Preparing the Project for Onboarding](docs/05-Multi-Tenancy-with-Flux/07-Preparing-the-Project-for-Onboarding.md)
- [Release Strategies with Flagger](docs/06-Release-Strategies-with-Flagger/)
  - [Installing Flagger](docs/06-Release-Strategies-with-Flagger/01-Installing-Flagger.md)
  - [Deploying the Nginx Ingress Controller](docs/06-Release-Strategies-with-Flagger/02-Deploying-the-Nginx-Ingress-Controller.md)
  - [Defining the Ingress Rule for Vote](docs/06-Release-Strategies-with-Flagger/03-Defining-the-Ingress-Rule-for-Vote.md)
  - [Changing Flux Code to Work with Flagger](docs/06-Release-Strategies-with-Flagger/04-Changing-Flux-Code-to-Work-with-Flagger.md)
  - [Enable Flagger Deployment](docs/06-Release-Strategies-with-Flagger/05-Enable-Flagger-Deployment.md)
  - [Triggering Blue/Green Deployment](docs/06-Release-Strategies-with-Flagger/06-Triggering-Blue-Green-Deployment.md)
