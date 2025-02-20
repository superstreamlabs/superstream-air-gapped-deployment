<div align="center">

![full logo superstream](https://github.com/superstreamlabs/superstream-engine/assets/107035359/19dc2e40-a907-49ee-9faf-d6d707633d53)

<b>Improve And Optimize Your Kafka In Literally Minutes.<br>
Reduce Costs and Boost Performance by 75% Without Changing a Single Component or Your Existing Kafka!</b>

</div>
This guide provides instructions for deploying the Superstream All-In-One distribution.

## Create Secrets with Randomly Generated Passwords for SSM
### The secret name `superstream-creds-control-plane` cannot be changed in the current release. This will be fixed in an upcoming release.
To create a secret for the Superstream with randomly generated passwords, run the following command:
```yaml
kubectl create secret generic superstream-creds-control-plane \
  --from-literal=postgres-password=$(openssl rand -base64 16 | tr -dc 'a-zA-Z0-9') \
  --from-literal=password=$(openssl rand -base64 16| tr -dc 'a-zA-Z0-9') \
  --from-literal=repmgr-password=$(openssl rand -base64 16| tr -dc 'a-zA-Z0-9') \
  --from-literal=admin-password=$(openssl rand -base64 16| tr -dc 'a-zA-Z0-9') \
  --from-literal=superstream-admin-password=$(openssl rand -base64 16| tr -dc 'a-zA-Z0-9') \
  --from-literal=control-plane-token=$(openssl rand -base64 48 | tr -dc 'a-zA-Z0-9' | head -c32) \
  --from-literal=encryption-secret-key=$(openssl rand -base64 48 | tr -dc 'a-zA-Z0-9' | head -c32) \
  --from-literal=jwt-secret-key=$(openssl rand -base64 48 | tr -dc 'a-zA-Z0-9' | head -c32) \
  --from-literal=jwt-api-secret-key=$(openssl rand -base64 48 | tr -dc 'a-zA-Z0-9' | head -c32) \
  -n superstream
```
### Note: The following keys should have a length of 32 characters:
 - encryption-secret-key
 - jwt-secret-key
 - jwt-api-secret-key
 - control-plane-token

## Configure Environment Tokens

For a more straightforward configuration, create a `custom_values.yaml` file and edit the following values:
```yaml
############################################################
# GLOBAL configuration for Superstream Engine
############################################################
global:
  engineName: ""                    # Define the superstream engine name within 32 characters, excluding '.', and using only lowercase letters, numbers, '-', and '_'.
  superstreamAccountId: ""          # Provide the account ID associated with the deployment, which could be used for identifying resources or configurations tied to a specific account.
  superstreamActivationToken: ""    # Enter the activation token required for services or resources that need an initial token for activation or authentication.
  skipLocalAuthentication: true
  onPrem: true                      
  ## If your environment uses a proxy server, uncomment the lines below and replace the URL with your proxy server's address.
  proxy:
    enabled: false
    proxyUrl: "https://your-proxy-server"


############################################################
# Superstream Control Plane config
############################################################
superstreamControlPlane:
# Configure the time when the samples will be taken (UTC)
# Default: 7:00 AM
  utcHourToTakeSamples: ""
############################################################
# NATS config
############################################################
# NATS HA Deployment. Default "true"
nats:
  config:
    cluster:
      enabled: true
# NATS storageClass configuration. The default is blank "".
    jetstream:
      fileStore:
        pvc:
          storageClassName: ""
    nats:
      port: 4222
      tls:
        enabled: false
        # set secretName in order to mount an existing secret to dir
        secretName: ""
        localCa:
          enabled: false
          secretName: ""               
############################################################
# Postgresql config
############################################################
postgresql-ha:
# Postgresql storageClass configuration. The default is blank "".
  persistence:
    enable: true    
    storageClass: ""          
############################################################
# Kafka Autoscaler config
############################################################
# Optional service to automatically scale the Kafka cluster up/down based on CPU and memory metrics  
autoScaler:
  enabled: true
```
## Proxy Configuration
If your environment requires a proxy server to connect to external services, set the global.proxy.enabled variable to true and provide the global.proxy.proxyUrl in the custom_values.yaml file. This configuration ensures that all critical services route traffic through the specified proxy. 
Additionally, make sure your proxy server permits connectivity to the following endpoints:

* **Prometheus:** https://prometheus.mgmt.superstream.ai
* **Loki:** https://loki.mgmt.superstream.ai
* **Stigg** https://api.stigg.io

## Deployment Instructions

To deploy the Superstream, run the following command:
```bash
helm repo add superstream-onprem https://k8s-onprem.superstream.ai/ --force-update && helm upgrade --install superstream superstream-onprem/superstream-onprem -f custom_values.yaml --create-namespace --namespace superstream --wait
```

## Configure valid FQDN records
To use the Superstream User Interface, the following two FQDN records should be exposed under the same domain.
 - Expose the Superstream Control Plane service. Using superstream-api at the beginning of the configured FQDN is a hard requirement. 
  Example: "superstream-api.example.com"
 - Expose the Superstream Control Plane UI service. 
  Example: superstream-app.example.com
 - Log in to the Superstream UI and connect your first Kafka cluster.

## Parameters
The following table lists the configurable parameters of the SuperStream chart and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.engineName`                                       | Define the superstream engine name within 32 characters, excluding '.', and using only lowercase letters, numbers, '-', and '_'. | `""`|
| `global.superstreamAccountId`                             | Provide the account ID associated with the deployment, which could be used for identifying resources or configurations tied to a specific account. | `""`|
| `global.superstreamActivationToken`                       | Enter the activation token required for services or resources that need an initial token for activation or authentication. | `""` |
| `global.skipLocalAuthentication`                          | Specifies whether to skip local authentication.                                      | `true`                             |
| `global.image.pullPolicy`                                 | Global image pull policy to use for all container images in the chart. Can be overridden by individual image pullPolicy. | `""` |
| `global.image.pullSecretNames`                            | Global list of secret names to use as image pull secrets for all pod specs in the chart. Secrets must exist in the same namespace. | `[]` |
| `global.image.registry`                                   | Global registry to use for all container images in the chart. Can be overridden by individual image registry. | `""` |
| `global.labels`                                   | Global labels to use for all container images in the chart. | `""` |
| `global.onPrem`                                           | Specifies if the deployment is for an on-premises environment. | `true` |
| `nats.config.cluster.enabled`                             | Indicates whether the NATS cluster is enabled.                                      | `true`                             |
| `nats.config.jetstream.fileStore.pvc.storageClassName`    | Specifies the storage class name for the Jetstream file store PVC.                  | `""`                               |
| `nats.config.nats.tls.enabled`                             | Enables or disables TLS (Transport Layer Security) for the NATS server. Set to `true` to enable TLS.                                      | `false`                             |
| `nats.config.nats.tls.secretName`                             | If provided, mounts an existing secret to the directory for TLS credentials. Useful for referencing pre-existing certificates and keys.                                      | `""`                             |
| `nats.config.nats.tls.localCa.enabled`                             | Enables or disables the use of a local Certificate Authority (CA) for generating TLS certificates.                                      | `false`                             |
| `nats.config.nats.tls.localCa.secretName`                             | The name of the secret containing the local CA’s certificates. Required if `nats.config.nats.tls.localCa.enabled` is set to `true`.                                      | `false`                             |
| `superstreamControlPlane.image.registry`                  |	Docker registry to use for pulling the control plane backend service images. | `""` |
| `superstreamControlPlane.secret.useExisting`              |	Determines whether to use an existing secret for the control plane.	| `true` |
| `superstreamControlPlane.service.port`                    |	Port for the control plane service.	| `8888` |
| `utcHourToTakeSamples`                                    |  Configure the time when the samples will be taken (UTC) | `""` |
| `superstreamControlPlane.extraEnv`                    |	A map of additional environment variables for the application.	| `{}` |
| `superstreamControlPlane.userInterface.image.registry`    |	Docker registry to use for pulling the control plane UI service images.	| `""` |
| `superstreamControlPlane.userInterface.extraEnv`                    |	A map of additional environment variables for the application.	| `{}` |
| `superstreamControlPlane.userInterface.service.port`      |	Port for the control plane UI service. | `80` |
| `superstreamEngine.releaseDate`                           | Release date for the backend component.                                             | `"2024-02-22-13-03"`               |
| `superstreamEngine.replicaCount`                          | Number of replicas for the backend deployment.                                      | `2`                                |
| `superstreamEngine.image.repository`                      | Docker image repository for the backend service.                                    | `superstreamlabs/superstream-data-plane-be` |
| `superstreamEngine.image.pullPolicy`                      | Policy for pulling the image.                                                       | `Always`                           |
| `superstreamEngine.image.tag`                             | Overrides the image tag.                                                            | `"latest"`                         |
| `superstreamEngine.imagePullSecrets`                      | Image pull secrets.                                                                 | `[]`                               |
| `superstreamEngine.nameOverride`                          | Overrides for Helm's default naming conventions.                                    | `""`                               |
| `superstreamEngine.fullnameOverride`                      | Full name override for Helm's default naming conventions.                           | `""`                               |
| `superstreamEngine.initContainers.image`                  | Image used for readiness checks or setup tasks before the main containers start.    | `curlimages/curl:8.6.0`            |
| `superstreamEngine.configMap.enable`                      | Enable ConfigMap settings.                                                          | `""`                               |
| `superstreamEngine.configMap.useExisting`                 | Determines whether to use an existing ConfigMap instead of creating a new one.      | `false`                            |
| `superstreamEngine.configMap.existingConfigMapName`       | Name of the existing ConfigMap to use if `useExisting` is `true`.                   | `my-existing-configmap`            |
| `superstreamEngine.configMap.name`                        | Name used when creating a new ConfigMap.                                            | `""`                               |
| `superstreamEngine.configMap.fileData.mountPath`          | Path where the ConfigMap will be mounted in the pod.                                | `/etc/superstream-conf/`           |
| `superstreamEngine.configMap.fileData.fileName`           | Name of the file to be used from the ConfigMap.                                     | `""`                               |
| `superstreamEngine.configMap.fileData.fileContent`        | Content of the file to be used from the ConfigMap.                                  | `""`                               |
| `superstreamEngine.secret.name`                           | Secret configuration for sensitive information.                                     | `superstream-creds`                |
| `superstreamEngine.secret.encryptionSecretKey`            | Encryption secret key used for sensitive information.                               | `""`                               |
| `superstreamEngine.secret.activationToken`                | Activation token for services or resources.                                         | `""`                               |
| `superstreamEngine.secret.useExisting`                    | Specifies whether to use an existing secret.                                        | `false`                            |
| `superstreamEngine.podAnnotations.prometheus.io/path`     | Path for Prometheus to scrape metrics from the pod.                                 | `"/monitoring/metrics"`            |
| `superstreamEngine.podAnnotations.prometheus.io/scrape`   | Specifies whether Prometheus should scrape metrics from the pod.                    | `'true'`                           |
| `superstreamEngine.podAnnotations.prometheus.io/port`     | Port for Prometheus to scrape metrics from the pod.                                 | `"7777"`                           |
| `superstreamEngine.podSecurityContext`                    | Security context settings for the pod.                                              | `{}`                               |
| `superstreamEngine.securityContext`                       | Security context for containers within the pod.                                     | `{}`                               |
| `superstreamEngine.serviceAccount.create`                 | Specifies whether a service account should be created.                              | `true`                             |
| `superstreamEngine.serviceAccount.annotations`            | Annotations to add to the service account.                                          | `{}`                               |
| `superstreamEngine.serviceAccount.name`                   | The name of the service account to use.                                             | `""`                               |
| `superstreamEngine.extraEnv`                    |	A map of additional environment variables for the application.	| `{}` |
| `superstreamEngine.service.enabled`                       | Enable service for the backend.                                                     | `true`                             |
| `superstreamEngine.service.type`                          | Type of service for the backend.                                                    | `ClusterIP`                        |
| `superstreamEngine.service.port`                          | Port for the backend service.                                                       | `7777`                             |
| `superstreamEngine.resources.limits.cpu`                  | CPU limit for the backend pod.                                                      | `8`                                |
| `superstreamEngine.resources.limits.memory`               | Memory limit for the backend pod.                                                   | `8Gi`                              |
| `superstreamEngine.resources.requests.cpu`                | CPU request for the backend pod.                                                    | `500m`                             |
| `superstreamEngine.resources.requests.memory`             | Memory request for the backend pod.                                                 | `1Gi`                              |
| `superstreamEngine.autoscaling.enabled`                   | Enable autoscaling for the backend.                                                 | `true`                             |
| `superstreamEngine.autoscaling.minReplicas`               | Minimum number of replicas for autoscaling.                                         | `2`                                |
| `superstreamEngine.autoscaling.maxReplicas`               | Maximum number of replicas for autoscaling.                                         | `5`                                |
| `superstreamEngine.autoscaling.targetCPUUtilizationPercentage` | CPU utilization percentage for autoscaling.                                         | `75`                               |
| `superstreamEngine.autoscaling.targetMemoryUtilizationPercentage` | Memory utilization percentage for autoscaling.                                      | `75`                               |
| `superstreamEngine.nodeSelector`                          | Node selectors to control the placement of pods.                                    | `{}`                               |
| `superstreamEngine.tolerations`                           | Tolerations for pods to tolerate certain node conditions or taints.                 | `[]`                               |
| `superstreamEngine.affinity`                              | Affinity rules for pod scheduling.                                                  | `{}`                               |
| `superstreamEngine.internalNatsConnection.host`           | Host for the internal NATS connection.                                              | `""`                               |
| `superstreamEngine.internalNatsConnection.port`           | Port for the internal NATS connection.                                              | `4222`                             |
| `superstreamEngine.controlPlane.host`                     | Host for the control plane connection.                                              | `"broker.superstream.dev"`         |
| `superstreamEngine.controlPlane.port`                     | Port for the control plane connection.                                              | `4222`                             |
| `superstreamEngine.syslog.enabled`                        | Determines whether the syslog is enabled for the superstream engine.                | `true`                             |
| `superstreamEngine.syslog.remoteSyslog`                   | Remote syslog server to send logs to.                                               | `"superstream-syslog"`             |
| `superstreamEngine.releaseDate`                           | Release date for the backend component.                                             | `"2024-03-20-11-12"`               |
| `autoScaler.enabled`                           | Enables the Kafka auto-scaler.                                             | `"true"`                |
| `autoScaler.releaseDate`                           | Release date for the autoscaler.                                             | `""`               |
| `autoScaler.replicaCount`                           | Enables the Kafka auto-scaler.                                             | `"false"`                |
| `autoScaler.image.repository`                           | Docker image repository for the Kafka auto-scaler.                                             | `"superstreamlabs/superstream-kafka-auto-scaler"`                |
| `autoScaler.image.pullPolicy`                           | Policy for pulling the Docker image.                                             | `Always`                |
| `autoScaler.image.tag`                           | Docker image tag (Overrides the image).                                             | `"latest"`                |
| `autoScaler.resources.limits.cpu`                           | CPU limit for the auto-scaler.                                             | `"2"`                |
| `autoScaler.resources.limits.memory`                           | Memory limit for the auto-scaler.                                             | `"2Gi"`                |
| `autoScaler.resources.requests.cpu`                           | CPU request for the auto-scaler.                                             | `"500m"`                |
| `autoScaler.resources.requests.memory`                           | Memory request for the auto-scaler.                                             | `"500Mi"`                |
| `autoScaler.autoscaling.enabled`                   | Enable autoscaling for the backend.                                                 | `true`                             |
| `autoScaler.autoscaling.minReplicas`               | Minimum number of replicas for autoscaling.                                         | `2`                                |
| `autoScaler.autoscaling.maxReplicas`               | Maximum number of replicas for autoscaling.                                         | `5`                                |
| `autoScaler.autoscaling.targetCPUUtilizationPercentage` | CPU utilization percentage for autoscaling.                                         | `75`                               |
| `autoScaler.autoscaling.targetMemoryUtilizationPercentage` | Memory utilization percentage for autoscaling.                                      | `75`                               |
| `autoScaler.extraEnv`                    |	A map of additional environment variables for the application.	| `{}` |
| `autoScaler.nodeSelector`                          | Node selectors to control the placement of pods.                                    | `{}`                               |
| `autoScaler.tolerations`                           | Tolerations for pods to tolerate certain node conditions or taints.                 | `[]`                               |
| `autoScaler.affinity`                              | Affinity rules for pod scheduling.                                                  | `{}`                               |
| `syslog.replicaCount`                                     | Number of replicas for the syslog deployment.                                       | `1`                                |
| `syslog.image.repository`                                 | Docker image repository for syslog.                                                 | `linuxserver/syslog-ng`            |
| `syslog.image.pullPolicy`                                 | Pull policy for the syslog image.                                                   | `IfNotPresent`                     |
| `syslog.image.tag`                                        | Tag for the syslog image.                                                           | `"4.5.0"`                          |
| `syslog.imagePullSecrets`                                 | Image pull secrets.                                                                 | `[]`                               |
| `syslog.service.type`                                     | Type of service for syslog.                                                         | `ClusterIP`                        |
| `syslog.service.port`                                     | Port for the syslog service.                                                        | `5514`                             |
| `syslog.service.protocol`                                 | Protocol used by the syslog server.                                                    | `UDP`                              |
| `syslog.extraEnv`                    |	A map of additional environment variables for the application.	| `{}` |
| `syslog.resources.limits.cpu`                             | CPU limit for the syslog pod.                                                       | `"100m"`                           |
| `syslog.resources.limits.memory`                          | Memory limit for the syslog pod.                                                    | `"256Mi"`                          |
| `syslog.resources.requests.cpu`                           | CPU request for the syslog pod.                                                     | `"50m"`                            |
| `syslog.resources.requests.memory`                        | Memory request for the syslog pod.                                                  | `"128Mi"`                          |
| `syslog.podAnnotations.prometheus.io/scrape`              | Specifies whether Prometheus should scrape metrics from the syslog pod.             | `'false'`                          |
| `syslog.remoteSyslog.destinationHost`                     | Destination host for the remote syslog.                                             | `telegraf`                         |
| `syslog.remoteSyslog.port`                                | Port for the remote syslog.                                                         | `6514`                             |
| `syslog.remoteSyslog.protocol`                            | Protocol (e.g., UDP) for the remote syslog.                                         | `udp`                              |
| `syslog.configMap.enabled`                                | Enable ConfigMap for syslog.                                                        | `true`                             |
| `syslog.configMap.name`                                   | Name of the ConfigMap for syslog.                                                   | `syslog-config`                    |
| `syslog.configMap.mountPath`                              | Mount path for the syslog ConfigMap.                                                | `/tmp/syslog-ng.conf`           |
| `syslog.configMap.subPath`                                | Specific file to mount from the ConfigMap.                                          | `syslog-ng.conf`                   |
| `syslog.persistence.enabled`                              | Enable persistence for syslog.                                                      | `false`                            |
| `syslog.persistence.size`                                 | Size of the persistent volume for syslog.                                           | `"1Gi"`                            |
| `syslog.persistence.accessModes`                          | Access modes (e.g., ReadWriteOnce) for the syslog persistent volume.                | `["ReadWriteOnce"]`                |
| `syslog.persistence.storageClassName`                     | Storage class name for the syslog persistent volume.                                | `"standard"`                       |
