# Generating ConfigMaps

Kustomize is also capable of generating objects to be used within the Kubernetes objects. In this case we can generate a CofigMap to be consumed by our vote application

```sh
cat >> deploy/vote/staging/kustomization.yaml << EOF

configMapGenerator:
  - name: vote-options
    envs:
      - options.env
EOF
```

Now we need to link the configMap to the Pod in the `deploy/vote/staging/deployment.yaml` file:

```sh
cat >> deploy/vote/staging/deployment.yaml << EOF
        envFrom:
        - configMapRef:
            name: vote-options
            optional: true
EOF
```

And finally generate the `options.env` file

```sh
cat >> deploy/vote/staging/options.env << EOF
OPTION_A=Rojo
OPTION_B=Azul
EOF
```

Verify the changes were applied via:

```sh
kustomize build deploy/vote/staging
# Here you'd be able to see a configMap has been generated and mentioned in the Deployment's pods.
```

# Commit and Promote your changes

```sh
git add deploy
git commit -m "feat: Add ConfigMap to override options in the vote site"
git push origin HEAD:refs/heads/main
```
