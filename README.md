# Substitute environment variables with Kustomize

The variable reference transformer can perform variable substitution in Kustomize. However, "vars are used to capture text from one resource's field and insert that text elsewhere." They are not intended to accept literal values or environment variables.

However, Kustomize's key value environment files support a second syntax in addition to `KEY=value`. If you [omit the value the environment variable will be used](https://github.com/kubernetes-sigs/kustomize/blame/6e7713281e716232481659fd5f602b66140c9a16/api/kv/kv.go#L163-L169). That is, specifying `KEY` without a value in a ConfigMap's env file will capture the build environment's `KEY` environment variable into the ConfigMap.

Now that the environment variables have been captured into a ConfigMap, it is possible to use them with variable reference transformer. If you are performing substitution in nonstandard places you will need to add additional entries to `varReference` in the transformer configuration.

Steps:

1. Create a `configMapGenerator` using an env file.
2. List all the needed environment variables in the env file _without a value_.
3. Define `vars` to extract each of the used environment variables out of the ConfigMap and into a Kustomize variable.
4. Add entries to transformer configuration if the default `varReference` list is not sufficient. Perhaps add `spec/source/path` for the Argo CD Application CRD.
5. Use environment variables as `$(FOO)` in your resource definitions.

See [`environment-variables/kustomization.yaml`](environment-variables/kustomization.yaml) for each above step and [`hello-world.yaml`](hello-world.yaml) for usage of the variables.

Local testing usage:
```
ARGOCD_APP_SOURCE_TARGET_REVISION=dev-123-feature-foo ENVIRONMENT=dev kustomize build .
```

If this is run from inside Argo CD (perhaps in an app of apps pattern) `ARGOCD_APP_SOURCE_TARGET_REVISION` should be automatically populated from the [Argo CD build environment](https://argoproj.github.io/argo-cd/user-guide/build-environment/). If you need additional variables (like `ENVIRONMENT`) set in Argo CD, you would need to run Argo CD itself with the additional environment variables configured. It is also possible to specify literal values like `ENVIRONMENT=dev` in the [`environment.env` file](environment-variables/environment.env).

Argo CD [does not appear to actually pass environment variables to Kustomize](https://github.com/argoproj/argo-cd/blob/d479d22de7e33dd5583bd51e4eb76163baf6318c/reposerver/repository/repository.go#L532-L538), so this probably doesn't work unless you run Kustomize via an [Argo CD config management plugin](https://argoproj.github.io/argo-cd/user-guide/config-management-plugins/).

Note: This does leave around an extra `ConfigMap` in your deployed environment called `environment` (the name is configurable). This is an unavoidable side effect of misusing a configMapGenerator.

Note 2: Missing environment variables will _not_ cause an error. Instead an empty string will be stored in the ConfigMap.

Note 3: I am aware of how horrifying this code is. 
