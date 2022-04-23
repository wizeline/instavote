# Create Source Controller

Add the Source Controller
```sh
flux create source git instavote \
  --url "https://github.com/${GITHUB_USER}/instavote.git" \
  --branch main \
  --interval 30s
```

Verify it was correclty added:


```sh
flux get sources git
```
