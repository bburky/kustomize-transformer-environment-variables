Usage:
```
ARGOCD_APP_SOURCE_TARGET_REVISION=dev-123 ENVIRONMENT=dev kustomize build .
```

If this is run from an Argo CD app of apps `ARGOCD_APP_SOURCE_TARGET_REVISION` should be automatically populated.
