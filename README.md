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
| cacerts.enabled | bool | `false` | Enable External Certificate Truststore |
| cacerts.filename | string | `"cacerts.dod"` | JKS cert store |
| certificateTrust.chain | string | `"-----BEGIN CERTIFICATE-----\nMIIDcDCCAligAwIBAgIRAK3oVrLXD6emb8k/a1J3K50wDQYJKoZIhvcNAQELBQAw\nSzEQMA4GA1UEChMHQWNtZSBDbzE3MDUGA1UEAxMuS3ViZXJuZXRlcyBJbmdyZXNz\nIENvbnRyb2xsZXIgRmFrZSBDZXJ0aWZpY2F0ZTAeFw0yNDEyMjYxNTAyMDdaFw0y\nNTEyMjYxNTAyMDdaMEsxEDAOBgNVBAoTB0FjbWUgQ28xNzA1BgNVBAMTLkt1YmVy\nbmV0ZXMgSW5ncmVzcyBDb250cm9sbGVyIEZha2UgQ2VydGlmaWNhdGUwggEiMA0G\nCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCruycRyEuB4a5lVAKuJG6InGhJl0hE\nxSKvsWeTKbDbqBkObtlTn+qD2gsh9UunQ6V1GI1CwlVG+sv818WcivBJsYaj8Mqq\nCdBil98Kxe1N6J6ua65CS4Ws9d7LWTvvXWTNQ/0t4JSt6W8L4+WXwipXEdjEyucp\n47zbRmg47Fg8eLDnsSI44cWZGZBI8PBgfUxCLMoqSb/iwA3XrdXjA3hQh43HO4mR\nKR/iCVIZ/bwmq7+RsVG6tzt8EKKuml+jKb2KoGVeTGqaaPsRbVd1e2KxqHwJvPaJ\ntM5Tl5mYWJ4VzgYQi/hIHkCicDFQqUi8sduRPWfX8mXghQFToJLu77UjAgMBAAGj\nTzBNMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMB\nAf8EAjAAMBgGA1UdEQQRMA+CDWluZ3Jlc3MubG9jYWwwDQYJKoZIhvcNAQELBQAD\nggEBAKlWSvxIqCwvO5gxQ2Ayw8LH+cYKBZMtq2+kQ7IthqJj2RZwCAR4+RKxlvcr\n/rgWNgk6x0873IG0sWYu1YORPrk35ysUwKLFogPfWFqAPc3SUcAGDgrVBOrKG9RB\nrY+Oj2klQsS68ih51/VCNFiapqDhqX58+mhblEumLsD+0B0ySDJrKGtPHZbTIzKR\nejznUBBDkNa3ZQ1Jn0rOK4YtmnwZVWiQ3bvxPAhg8HUbe81A+ZuWZ3iE/eWFNhpn\nKicBYgK3TBPXEu1xpQxu8pcNC+1Np3yvF7dbI806Pohlyr3oZpkE1jcVkrLLyB5Y\n1P38z8hjD+aOcacGbj2quTXFkQg=\n-----END CERTIFICATE-----\n"` |  |
| certificateTrust.storePassword | string | `nil` |  |
| fullnameOverride | string | `""` | Override deployment name |
| guacamole.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacamole.image.repository | string | `"guacamole/guacamole"` | Image repository |
| guacamole.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacamole.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","memory":"1Gi"}}` | Pod assigned resources |
| guacamole.service.port | int | `8080` | Service port number |
| guacamole.service.type | string | `"ClusterIP"` | Service type |
| guacamole.settings | object | "" | Key-value settings directly passed as environment variables for guacamole configuration |
| guacd.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacd.image.repository | string | `"guacamole/guacd"` | Image repository |
| guacd.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacd.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","memory":"1Gi"}}` | Pod assigned resources |
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
| postgres.service.port | string | `"5432"` | Service port number |
| postgres.service.type | string | `"ClusterIP"` | Service type |
| postgres.user | string | `"guacamole"` | Postgres username |
| replicaCount | int | `1` | Replica pairs to use for each Guacd and Guacamole pair |
| securityContext | object | `{}` |  |
