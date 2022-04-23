# Deploying Vote Result with ChartSource

Besides the Source as `HelmRepository` source we're also capable of deploying a
`HelmRelease` with a `GitRepository`. Lets deploy the vote-result application
with this technique:

## Generate a helm chart for vote-result

First we need to install the helm CLI:

```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Now we can use the default template in the `instavote` repository:

```sh
cd instavote
mkdir -p deploy/charts && cd $_
helm create result

# A template for a Chart should be generated
ls -R result
```

## Update the values to reflect the result-vote application

Update the settings for `image.repository`, `image.tag` and `service.type`
```sh
sed -i '' \
  -e 's@repository:.*@repository: schoolofdevops/vote-result@' \
  -e 's@tag:.*@tag: "latest"@' \
  -e 's@type: ClusterIP@type: NodePort@' \
  result/values.yaml
```

Commit and Expose the values:

```sh
git add result
git commit -am "feat: Add result chart definition"
git push origin HEAD:refs/heads/main
```
