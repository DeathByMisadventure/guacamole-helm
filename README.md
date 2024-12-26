# Guacamole Helm Chart

This helm chart will deploy pairs of guacamole and guacd containers in Kubernetes for remote access into the environment.

![Version: 1.5.3](https://img.shields.io/badge/Version-1.5.3-informational?style=flat-square) ![AppVersion: 1.5.3](https://img.shields.io/badge/AppVersion-1.5.3-informational?style=flat-square)

## Prerequisites

* The Postgresql server for guacamole must be deployed into the environment.
* AKS nodes must be able to reach the subnets the VMs are deployed in to.

## Configure values.yaml or valuesoverride.yaml

A few variables need to be created in the values.yaml.  Namely postgresql connection information, OpenID, and if a custom root certificate will be used.

## Deployment

Create a namespace:  `kubectl create ns guacamole`

Add the secret for the certificate if using an ingress

Install the helm chart with:  `helm install --namespace guacamole guacamole .`

## Creating a DoD JKS Trust Store

1. Go to the Tools section on the [Public Cyber Mil](https://public.cyber.mil/pki-pke/end-users/getting-started/) website.
2. Click the Windows option.
3. Find the link to download the latest version of the Non Administrator InstallRoot tool.
4. Download the installer that is appropriate for the platform you are using.
5. Double-click the downloaded file to start the installation process.
6. On the Welcome screen, click Next.
7. Specify the directory in which you want to install the tool, and click Next.
8. On the InstallRoot Features screen, click Next.
9. On the next screen, click Install.
10. After the installation is finished, click Run InstallRoot.
11. Expand DoD.
12. Press Shift and select the list of certificates.
13. Click the Certificate tab, and and then click PEM.
14. In the Export dialog box, specify the location in which you want to save the exported files.
15. From the command line, navigate to the directory in which you saved the exported files.
16. Run this command after installing:

  ```powershell
  @ECHO OFF for /r %%f in (*.cer) do ( "keytool.exe" -importcert -noprompt -file "%%f" -alias "%%~nf" - keystore DoDRoot.jks -storepass "changeit" -keypass "changeit" )
  ```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| cacerts.enabled | bool | `false` | Enable External Certificate Truststore |
| cacerts.filename | string | `"cacerts.dod"` | JKS cert store |
| fullnameOverride | string | `""` | Override deployment name |
| guacamole.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacamole.image.repository | string | `"guacamole/guacamole"` | Image repository |
| guacamole.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacamole.resources | object | `{}` | Pod assigned resources |
| guacamole.service.port | int | `8080` | Service port number |
| guacamole.service.type | string | `"ClusterIP"` | Service type |
| guacamole.settings | object | "" | Key-value settings directly passed as environment variables for guacamole configuration |
| guacd.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| guacd.image.repository | string | `"guacamole/guacd"` | Image repository |
| guacd.image.tag | string | `"{{ .Chart.AppVersion }}"` | Image tag defaults to Chart AppVersion |
| guacd.resources | object | `{}` | Pod assigned resources |
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
| postgres.resources | object | `{}` | Pod assigned resources |
| postgres.service.port | string | `"5432"` | Service port number |
| postgres.service.type | string | `"ClusterIP"` | Service type |
| postgres.user | string | `"guacamole"` | Postgres username |
| replicaCount | int | `1` | Replica pairs to use for each Guacd and Guacamole pair |
| securityContext | object | `{}` |  |
