# Guacamole with Postgresql Helm Chart

Apache Guacamole is a clientless remote desktop gateway. It supports standard protocols like VNC, RDP, and SSH. This version of the helm chart includes support for custom configuration such as OpenID and custom CA root certificate capabilities. This helm chart will deploy pairs of guacamole and guacd containers in Kubernetes for remote access into your secured environment.

![Version: 1.6.0](https://img.shields.io/badge/Version-1.6.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.6.0](https://img.shields.io/badge/AppVersion-1.6.0-informational?style=flat-square)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=DeathByMisadventure_guacamole-helm&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=DeathByMisadventure_guacamole-helm)

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

## Enabling INGRESS-NGINX

If you do not currently have an ingress solution, setting ingress-nginx.enabled to true will import it into the
kubernetes environment.  If you do have an ingress solution in the cluster, then it may remain disabled.

## NGINX Modsecurity OWASP Web Application Firewall (WAF)

`ingress-nginx.controller.config` in the values.yaml configuration enables the [Web Application Firewall (WAF)](https://kubernetes.github.io/ingress-nginx/user-guide/third-party-addons/modsecurity/).  The Ingress WAF comes with the [OWASP Core Rule Set (CRS)](https://owasp.org/www-project-modsecurity-core-rule-set/).

The OWASP CRS provides the rules for the NGINX ModSecurity WAF to block SQL Injection (SQLi), Remote Code Execution (RCE), Local File Include (LFI), cross‑site scripting (XSS), and many other attacks.

The necessary rule changes to work with Guacamole have been applied.

Two solutions exist:
* If using the ingress-nginx dependency, then the values.yaml has this enabled by default.
* If ingress-nginx is disabled, then you can optionally enable it, depending on your ingress solution. If ingress-nginx then
ensure that the allowSnippetAnnotations is enabled when it was deployed.

## Deployment

Import the dependencies for the helm chart:

`helm dependency update`

Create a namespace if needed:

`kubectl create ns guacamole`

If using the internal Postgres deployment, create the PV and deploy it.

Customize any values by creating a local-values.yaml and including the overrides from the main values.yaml.

Add a secret for the ingress TLS certificate if needed, and configure TLS in the values.
[Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/)

`kubectl create --namespace guacamole secret tls external-tls-cert --cert=tls.crt --key=tls.key`

Install the helm chart with:

`helm install --namespace guacamole -f ./local-values.yaml deployment-name .`

## Values

| Key | Type | Description |
|-----|------|-------------|
| certificateTrust.chain | string | Chain of 0 to many PEM certificates which will be imported into the trust store |
| certificateTrust.storePassword | string | Override the JKS store password, if blank the password defaults to 'changeit' |
| credentials.hash | string | Guacadmin password hash |
| credentials.salt | string | Guacadmin password salt |
| fullnameOverride | string | Override deployment name |
| guacamole.image.pullPolicy | string | Image pull policy |
| guacamole.image.repository | string | Image repository |
| guacamole.image.tag | string | Image tag defaults to Chart AppVersion |
| guacamole.name | string | Container Name |
| guacamole.resources | object | Pod assigned resources |
| guacamole.securityContext | object | Pod security context |
| guacamole.service.port | int | Service port number |
| guacamole.service.type | string | Service type |
| guacamole.settings | object | Key-value settings directly passed as environment variables for guacamole configuration |
| guacd.image.pullPolicy | string | Image pull policy |
| guacd.image.repository | string | Image repository |
| guacd.image.tag | string | Image tag defaults to Chart AppVersion |
| guacd.name | string | Container Name |
| guacd.resources | object | Pod assigned resources |
| guacd.securityContext | object | Pod security context |
| guacd.service.port | int | Service port number |
| guacd.service.type | string | Service type |
| imagePullSecrets | list | Image pull secrets |
| ingress-nginx.controller.config | object | Ingress-NGINX Configuration for MODSECURITY OWASP Protection Comment out this entire section to disable the WAF |
| ingress-nginx.enabled | bool | Enable the ingress-nginx module dependency if not enabled in the cluster |
| ingress.annotations | object | Ingress annotations |
| ingress.className | string | Ingress class type |
| ingress.enabled | bool | Enable Ingress |
| ingress.host | string |  |
| ingress.tls | string | Enable TLS |
| nameOverride | string | Override deployment name |
| postgres.database | string | Database name |
| postgres.enabled | bool | Enable internal postgres database |
| postgres.hostname | string | If internal postgres is disabled, the external database hostname |
| postgres.image.pullPolicy | string | Image pull policy |
| postgres.image.repository | string | Image repository |
| postgres.image.tag | string | Image tag |
| postgres.name | string | Container Name |
| postgres.password | string | Postgres password |
| postgres.pvc.selector | string | Selector to match pre-provisioned PV |
| postgres.pvc.storageClassName | string | Storage Class Name for a pre-provisioned PV |
| postgres.pvc.storageRequest | string | Postgres PVC storage request size |
| postgres.replicas | int | Number of replicas |
| postgres.resources | object | Pod assigned resources |
| postgres.securityContext | string | Pod security context |
| postgres.service.port | string | Service port number |
| postgres.user | string | Postgres username |
| replicaCount | int | Replica pairs to use for each Guacd and Guacamole pair |
