# Verify Controllers Installed

## Retrieve the SourceControllers

```sh
flux get sources git
```

## Retrieve the Kustomizations:

```sh
flux get kustomizations
```


----

## Verify the Deployed Repository Components

```sh
☁  flux-infra [main] tree
.
└── clusters
    └── dev
        └── flux-system
            ├── gotk-components.yaml
            ├── gotk-sync.yaml
            └── kustomization.yaml
```
