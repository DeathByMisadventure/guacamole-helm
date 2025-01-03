# Guacamole with Postgresql Helm Chart

Apache Guacamole is a clientless remote desktop gateway. It supports standard protocols like VNC, RDP, and SSH. This version of the helm chart includes support for custom configuration such as OpenID and custom CA root certificate capabilities. This helm chart will deploy pairs of guacamole and guacd containers in Kubernetes for remote access into your secured environment.

![Version: 1.5.5](https://img.shields.io/badge/Version-1.5.5-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.5.5](https://img.shields.io/badge/AppVersion-1.5.5-informational?style=flat-square)

**Homepage:** <https://github.com/DeathByMisadventure/guacamole-helm>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://kubernetes.github.io/ingress-nginx | ingress-nginx | ~4.11.3 |

## Prerequisites

A Postgresql server for Guacamole must be deployed into the environment, or configure
the values to enable the built in Postgresql instance.  For enterprise use, an external
Postgresql server is recommended.

## Configure values.yaml or valuesoverride.yaml

A few variables need to be created in the values.yaml or in a values override. Documentation
of the variables are below or in the values.yaml file.

Optional configuration for Guacamole can be configured under guacamole.settings: such as
OpenID authentication.

All environmental variable names will automatically be uppercased and properly underscored.

### OpenID Configuration

For OpenID, this configuration block under settings will be automatically imported as the
environmental variables.

This example is for configuring this Guacamole instance with [Azure Entra ID](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc).

```yaml
guacamole:
  settings:
    openid_authorization_endpoint: 'https://login.microsoftonline.com/${AZURE_TENANT_ID}/oauth2/v2.0/authorize'
    openid_jwks_endpoint: 'https://login.microsoftonline.com/${AZURE_TENANT_ID}/discovery/v2.0/keys'
    openid_issuer: 'https://login.microsoftonline.com/${AZURE_TENANT_ID}/v2.0'
    openid_client_id: '${OPENID_CLIENT_ID}'
    openid_redirect_uri: 'https://${GUACAMOLE_URL}'
    openid_username_claim_type: 'email'
    openid_scope: 'openid email profile'
    extension_priority: '*, openid'
```

Keycloak authentication source would be similar.

```yaml
guacamole:
  settings:
    openid_authorization_endpoint: https://${KEYCLOAK_URL}/realms/${KEYCLOAK_REALM}/protocol/openid-connect/auth
    openid_jwks_endpoint: https://${KEYCLOAK_URL}/realms/${KEYCLOAK_REALM}/protocol/openid-connect/certs
    openid_issuer: https://${KEYCLOAK_URL}/realms/${KEYCLOAK_REALM}
    openid_client_id: '${OPENID_CLIENT_ID}'
    openid_redirect_uri: 'https://${GUACAMOLE_URL}'
    openid_username_claim_type: 'preferred_username'
    openid_scope: 'openid email profile'
    extension_priority: '*, openid'
```

For a SAML provider such as [Azure Entra ID](https://learn.microsoft.com/en-us/entra/architecture/auth-saml)

```yaml
guacamole:
  settings:
    saml-idp-metadata-url: 'https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml'
    saml-entity-id: 'https://${GUACAMOLE_URL}'
    saml-callback-url: 'https://${GUACAMOLE_URL}'
    extension-priority: '*, saml'
```

### Custom Certificate Authority Trust

This chart can automatically create a JKS trust store of a private CA certificate chain.  This is useful
in the scenario where a private Keycloak or Authentik identity management service is used in your environment,
and it utilizes a private Certificate Authority.

Simply create all necessary PEM certificates for the certificate root and any intermediate certificates.  Add them under
certificateTrust.chain: and an init container will be created to import them properly into the Guacamole container.

## Use Internal Postgresql

If you utilize the instance of Postgresql included in this chart, a Persistent Volume (PV) is recommended.
Create a PV according to the requirements of your environment.

This demo kubernetes PV will create with storage in your kubernetes host's /tmp/postgres-pv directory.
Remember to deploy this in the same namespace as the helm deployment.

The two keys are the `storageClassName:` and the `metadata.name:` which will then need to match
`postgres.pvc.storageClassName:` and the `postgres.pvc.selector:` in values.yaml for the Persistent Volume Claim
used in the helm chart.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /tmp/postgres-pv
```

## NGINX Modsecurity OWASP

`ingress-nginx.controller.config` in the values.yaml configuration enables the [OWASP Web Application Firewall](https://kubernetes.github.io/ingress-nginx/user-guide/third-party-addons/modsecurity/).

The necessary rule changes to work with Guacamole have been applied.

## Deployment

Import the dependencies for the helm chart:

`helm dependency update`

Create a namespace if needed:

`kubectl create ns guacamole`

Add a secret for the ingress TLS certificate if needed, and configure TLS in the values.
[Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/)

`kubectl create --namespace guacamole secret tls external-tls-cert --cert=tls.crt --key=tls.key`

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
| guacamole.name | string | `"guacamole"` | Container Name |
| guacamole.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| guacamole.securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":false,"seccompProfile":{"type":"RuntimeDefault"}}` | Pod security context |
| guacamole.service.port | int | `8080` | Service port number |
| guacamole.service.type | string | `"ClusterIP"` | Service type |
| guacamole.settings | object | "" | Key-value settings directly passed as environment variables for guacamole configuration |
| guacd.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacd.image.repository | string | `"guacamole/guacd"` | Image repository |
| guacd.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacd.name | string | `"guacd"` | Container Name |
| guacd.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| guacd.securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":true,"seccompProfile":{"type":"RuntimeDefault"}}` | Pod security context |
| guacd.service.port | int | `4822` | Service port number |
| guacd.service.type | string | `"ClusterIP"` | Service type |
| imagePullSecrets | list | `[]` | Image pull secrets |
| ingress-nginx.controller.config | object | `{"enable-modsecurity":"true","enable-owasp-modsecurity-crs":"true","modsecurity-snippet":"SecRuleEngine On\nSecStatusEngine Off\nSecAuditLog /dev/stdout\nSecAuditLogFormat JSON\nSecAuditLogParts ABCFHKZ\nSecAuditEngine RelevantOnly\nSecPcreMatchLimit 500000\nSecPcreMatchLimitRecursion 500000\nSecAction \"id:900200,phase:1,nolog,pass,t:none,setvar:tx.allowed_methods=GET HEAD POST OPTIONS PUT PATCH DELETE\"\nSecRuleRemoveById 920440\n","modsecurity-transaction-id":"$request_id"}` | Ingress-NGINX Configuration for MODSECURITY OWASP Protection |
| ingress.annotations | string | `nil` | Ingress annotations |
| ingress.className | string | `"nginx"` | Ingress class type |
| ingress.enabled | bool | `true` | Enable Ingress |
| ingress.hosts[0] | object | `{"host":"guac.localdev.me","paths":["/"]}` | Ingress hostname to bind |
| ingress.hosts[0].paths | list | `["/"]` | Ingress host paths |
| ingress.tls | string | `nil` | Enable TLS |
| nameOverride | string | `""` | Override deployment name |
| postgres.database | string | `"guacamole"` | Database name |
| postgres.enabled | bool | `true` | Enable internal postgres database |
| postgres.hostname | string | `"postgres.localdev.me"` | If internal postgres is disabled, the external database hostname |
| postgres.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| postgres.image.repository | string | `"postgres"` | Image repository |
| postgres.image.tag | string | `"16-alpine"` | Image tag |
| postgres.name | string | `"postgres"` | Container Name |
| postgres.password | string | `"password"` | Postgres password |
| postgres.pvc.selector | string | `nil` | Selector to match pre-provisioned PV |
| postgres.pvc.storageClassName | string | `nil` | Storage Class Name for a pre-provisioned PV |
| postgres.pvc.storageRequest | string | `"100Mi"` | Postgres PVC storage request size |
| postgres.replicas | int | `1` | Number of replicas |
| postgres.resources | object | `{"limits":{"cpu":"100m","memory":"1Gi"},"requests":{"cpu":"100m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| postgres.securityContext | string | `nil` | Pod security context |
| postgres.service.port | string | `"5432"` | Service port number |
| postgres.service.type | string | `"ClusterIP"` | Service type |
| postgres.user | string | `"guacamole"` | Postgres username |
| replicaCount | int | `1` | Replica pairs to use for each Guacd and Guacamole pair |
