# Defining a Helm Repository as a Source

Similarly as with the Source controllers for Git, we need to generate a source
for Helm in the infrastructure repository:

```sh
cd flux-infra

# Preview the output of the HelmRepository controller
flux create source helm helm-charts \
  --url=https://wizeline.github.io/helm-charts/ \
  --export

# Now execute the command so the Source controller gets added
flux create source helm helm-charts --url=https://wizeline.github.io/helm-charts/

# Verify the Controller is available
flux get sources helm helm-charts

# Save the changes to a file
flux create source helm helm-charts \
  --url=https://wizeline.github.io/helm-charts/ \
  --export > clusters/staging/helm-charts-helmrepository.yaml
```

Now the the controller is visible we can commit and publish our changes

```sh
git add clusters/staging/
git commit -am "feat: Add HelmRepository in staging"
git push origin HEAD:refs/heads/main
```
