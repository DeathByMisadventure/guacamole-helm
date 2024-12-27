# Guacamole Helm Chart

This helm chart will deploy pairs of guacamole and guacd containers in Kubernetes for remote access into the environment.

![Version: 1.5.5](https://img.shields.io/badge/Version-1.5.5-informational?style=flat-square) ![AppVersion: 1.5.5](https://img.shields.io/badge/AppVersion-1.5.5-informational?style=flat-square)

## Prerequisites

* The Postgresql server for guacamole must be deployed into the environment, or configure
the values to enable the built in Postgresql instance.

## Configure values.yaml or valuesoverride.yaml

A few variables need to be created in the values.yaml or in a values override.

## Deployment

Create a namespace:

`kubectl create ns guacamole`

Add the secret for the certificate if using an ingress

Install the helm chart with:

`helm install --namespace guacamole guacamole .`

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| certificateTrust.chain | string | "" | Chain of 0 to many PEM certificates which will be imported into the trust store |
| certificateTrust.storePassword | string | `nil` | Override the JKS store password, if blank the password defaults to 'changeit' |
| fullnameOverride | string | `""` | Override deployment name |
| guacamole.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacamole.image.repository | string | `"guacamole/guacamole"` | Image repository |
| guacamole.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacamole.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","memory":"1Gi"}}` | Pod assigned resources |
| guacamole.securityContext | string | `nil` | Pod security context |
| guacamole.service.port | int | `8080` | Service port number |
| guacamole.service.type | string | `"ClusterIP"` | Service type |
| guacamole.settings | object | "" | Key-value settings directly passed as environment variables for guacamole configuration |
| guacd.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacd.image.repository | string | `"guacamole/guacd"` | Image repository |
| guacd.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacd.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","memory":"1Gi"}}` | Pod assigned resources |
| guacd.securityContext | string | `nil` | Pod security context |
| guacd.service.port | int | `4822` | Service port number |
| guacd.service.type | string | `"ClusterIP"` | Service type |
| imagePullSecrets | list | `[]` | Image pull secrets |
| ingress.annotations | object | `{}` | Ingress annotations |
| ingress.className | string | `"nginx"` | Ingress class type |
| ingress.enabled | bool | `true` | Enable Ingress |
| ingress.hosts[0] | object | `{"host":"guac.localdev.me","paths":["/"]}` | Ingress hostname to bind |
| ingress.hosts[0].paths | list | `["/"]` | Ingress host paths |
| ingress.tls | list | `[]` | Enable TLS |
| nameOverride | string | `""` | Override deployment name |
| postgres.database | string | `"guacamole"` | Database name |
| postgres.enabled | bool | `true` | Enable internal postgres database |
| postgres.hostname | string | `"postgres.localdev.me"` | If internal postgres is disabled, the external database hostname |
| postgres.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| postgres.image.repository | string | `"postgres"` | Image repository |
| postgres.image.tag | string | `"16-alpine"` | Image tag |
| postgres.password | string | `"password"` | Postgres password |
| postgres.pvc.claim0.storageRequest | string | `"100Mi"` | Postgres PVC storage request size |
| postgres.replicas | int | `1` | Number of replicas |
| postgres.resources | object | `{"limits":{"cpu":"100m","memory":"1Gi"},"requests":{"cpu":"100m","memory":"1Gi"}}` | Pod assigned resources |
| postgres.securityContext | string | `nil` | Pod security context |
| postgres.service.port | string | `"5432"` | Service port number |
| postgres.service.type | string | `"ClusterIP"` | Service type |
| postgres.user | string | `"guacamole"` | Postgres username |
| replicaCount | int | `1` | Replica pairs to use for each Guacd and Guacamole pair |
