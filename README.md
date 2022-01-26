# Deploying Solid Community Server
## TL;DR
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release css
## Introduction
This chart bootstraps a [Solid Community Server](https://github.com/solid/community-server) deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites
Kubernetes 1.19+
Helm 3.2.0+
PV provisioner support in the underlying infrastructure if you want to use Persistence functionality.

## Installing the Chart
To install the chart with the release name my-release:

```
git clone https://github.com/idlab-gent/css-helm-chart.git
cd css-helm-chart
helm install my-release css
```

These commands deploy Solid Community Server on the Kubernetes cluster in the default configuration. The Parameters section lists the parameters that can be configured during installation.

> Tip: List all releases using `helm list`

## Uninstalling the Chart
To uninstall/delete the my-release deployment:

```
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Workarounds for standing issues
Use `baseUrlOverride` to set it to `http://localhost:3000/`, deploy and use port-forwarding to connect to your Solid App over localhost.
`kubectl port-forward <css-pod> 3000:3000`

## Parameters

### Image parameters

| Name                | Description                                      | Value                           |
| ------------------- | ------------------------------------------------ | ------------------------------- |
| `image.registry`    | CSS image registry                               | `docker.io`                     |
| `image.repository`  | CSS image repository                             | `solidproject/community-server` |
| `image.tag`         | CSS image tag (immutable tags are recommended)   | `2.0.1`                         |
| `image.pullPolicy`  | CSS image pull policy                            | `IfNotPresent`                  |
| `image.pullSecrets` | Specify docker-registry secret names as an array | `[]`                            |


### Common parameters

| Name               | Description                               | Value |
| ------------------ | ----------------------------------------- | ----- |
| `nameOverride`     | String to partially override css.fullname | `""`  |
| `fullnameOverride` | String to fully override css.fullname     | `""`  |


### Solid Community Server parameters

| Name                   | Description                                                                                                                                                         | Value     |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- |
| `config.bundled`       | Use one of the bundled configs (options: default, file, sparlql-endpoint, etc.)                                                                                     | `default` |
| `config.configMapName` | Name of configMap holding a custom css config                                                                                                                       | `""`      |
| `config.configMapKey`  | Key for the configfile to be used from the configMap                                                                                                                | `""`      |
| `logLevel`             | Log level: silly, debug, verbose, info, warn, error                                                                                                                 | `info`    |
| `showStacktrace`       | Enables detailed logging on error pages.                                                                                                                            | `false`   |
| `baseUrlOverride`      | From the helm config, an appropriate --baseUrl value will be passed to the community server. If however you wish to override this set this parameter appropriately. | `""`      |


### Persistence parameters

| Name                        | Description                                                                         | Value   |
| --------------------------- | ----------------------------------------------------------------------------------- | ------- |
| `persistence.enabled`       | Enable persistence on CSS using a `PersistentVolumeClaim`. If false, use emptyDir   | `false` |
| `persistence.existingClaim` | Name of an existing `PersistentVolumeClaim` for CSS replicas                        | `""`    |
| `persistence.subPath`       | Subdirectory of the volume to mount at (mostly useful when using an existing claim) | `""`    |
| `persistence.storageClass`  | CSS persistent volume storage Class                                                 | `""`    |
| `persistence.size`          | CSS persistent volume size                                                          | `128Mi` |
| `persistence.selector`      | Selector to match an existing Persistent Volume                                     | `{}`    |


### Kubernetes Service parameters

| Name               | Description                                                                     | Value       |
| ------------------ | ------------------------------------------------------------------------------- | ----------- |
| `service.type`     | CSS Kubernetes service type                                                     | `ClusterIP` |
| `service.port`     | CSS Kubernetes service port                                                     | `80`        |
| `service.nodePort` | CSS Kubernetes service node port, only relevant when service.type == `NodePort` | `""`        |


### Ingress resource parameters

| Name                       | Description                                                                                                                      | Value       |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| `ingress.enabled`          | Set to true to enable ingress record generation                                                                                  | `false`     |
| `ingress.host`             | When the ingress is enabled, a host pointing to this will be created                                                             | `CSS.local` |
| `ingress.path`             | Default path for the ingress resource                                                                                            | `/`         |
| `ingress.annotations`      | Additional annotations for the Ingress resource. To enable certificate autogeneration, place here your cert-manager annotations. | `{}`        |
| `ingress.tls`              | TLS Configuration                                                                                                                | `[]`        |
| `ingress.ingressClassName` | IngressClass that will be be used to implement the Ingress (Kubernetes 1.18+)                                                    | `""`        |


### Infrastructure parameters

| Name                 | Description                                | Value |
| -------------------- | ------------------------------------------ | ----- |
| `resources.limits`   | The resources limits for CSS containers    | `{}`  |
| `resources.requests` | The requested resources for CSS containers | `{}`  |
| `podAnnotations`     | CSS Pod annotations                        | `{}`  |
| `securityContext`    | Security Context for CSS Container         | `{}`  |
| `podSecurityContext` | Security Context for CSS Pod               | `{}`  |
| `nodeSelector`       | Node labels for pod assignment             | `{}`  |
| `tolerations`        | Tolerations for pod assignment             | `[]`  |
| `affinity`           | Affinity for pod assignment                | `{}`  |


Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,
```
helm install my-release \
  --set config.bundled=file \
  css
```
The above command deploys Solid Community Server with the bundled `file` config.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```
helm install my-release -f values.yaml css
```
> Tip: You can use the default values.yaml

You can even mix these two modes to override values from your YAML file with the `--set` argument. For example,

```
helm install my-release -f values.yaml --set config.bundled=file css
```
> This can be useful when trying out config changes or to enable debug logging for temporarily.

# Configuration and Installation details

## Using a bundled config
To use any of the bundled configs of CSS, specify which config to use by overriding `config.bundled` with any of the following values:
- default (default)
- dynamic
- example-https-file
- file
- file-no-setup
- memory-subdomains
- path-routing
- restrict-idp
- sparql-endpoint-no-setup
- sparql-endpoint
- sparql-file-storage

## Using a custom config
To use a custom config, the `config.configMapName` and `config.configMapKey` variables are used.

First create a configmap resource to hold your config on the cluster. For example when you have a json file called `my-config.json`:
```
kubectl create configmap my-configmap --from-file my-config.json
```

Alternatively when you have multiple configs in a directory called `my-configs`, you can bundle a whole directory into one configmap like so:
```
kubectl create configmap my-configmap --from-file my-configs/
```

Then override this name and key values accordingly (key of the config will be the same as the filename), for example:
```yaml
config:
    configMapName: my-configmap
    configMapKey: my-config.json
```