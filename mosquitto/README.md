# Mosquitto

[Eclipse Mosquitto](https://mosquitto.org) is an open source message broker that implements the MQTT (MQ Telemetry Transport) protocol v3.1


## TL;DR;

```bash
$ helm repo add queiroz https://charts.queiroz.dev
$ helm install queiroz/mosquitto
```

## Introduction

This chart bootstraps a [mosquitto](https://hub.docker.com/_/eclipse-mosquitto) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.6+ with Beta APIs enabled
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `mosquitto`:

```bash
helm install --name=mosquitto queiroz/mosquitto \
    --namespace=broker \
    --set persistence.enabled=true \
    --set persistence.storageClass=longhorn \
    --set ingress.enabled=true \
    --set ingress.tls=true \
    --set ingress.secretName=mosquitto-ingress-tls \
    --set ingress.hostname=mosquitto.broker.queiroz.dev \
    --set ingress.annotations."certmanager\.k8s\.io/cluster-issuer"="letsencrypt-prod" \
    --set ingress.annotations."kubernetes\.io/ingress"="nginx" \
    --set ingress.annotations."kubernetes\.io/tls-acme"="true" \
    --set ingress.annotations."kubernetes\.io/ingress.allow-http"="false" \
    --set ingress.annotations."nginx\.ingress\.kubernetes\.io/proxy-connect-timeout"="30" \
    --set ingress.annotations."nginx\.ingress\.kubernetes\.io/proxy-read-timeout"="1800" \
    --set ingress.annotations."nginx\.ingress\.kubernetes\.io/proxy-send-timeout"="1800"
```

The command deploys mosquitto on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `mosquitto` deployment:

```bash
$ helm delete mosquitto
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the mosquitto chart and their default values.

|         Parameter          |                Description                 |                   Default                  |
|----------------------------|--------------------------------------------|--------------------------------------------|
| `image`                    | mosquitto image                            | `eclipse-mosquitto:{VERSION}`              |
| `imagePullPolicy`          | Image pull policy.                         | `IfNotPresent`                             |
| `persistence.enabled`      | Use a PVC to persist data                  | `true`                                     |
| `persistence.storageClass` | Storage class of backing PVC               | `standard`                                 |
| `persistence.accessMode`   | Use volume as ReadOnly or ReadWrite        | `ReadWriteOnce`                            |
| `persistence.size`         | Size of data volume                        | `2Gi`                                      |
| `resources`                | CPU/Memory resource requests/limits        | Memory: `100Mi`, CPU: `50m`                |
| `config`                   | Multi-line string                          | see below                                  |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install --name mosquitto \
  --set imagePullPolicy=Always queiroz/mosquitto
```

The above command sets the mosquitto imagePullPolicy to `Always`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install --name mosquitto -f values.yaml queiroz/mosquitto
```

> **Tip**: You can use the default [values.yaml](values.yaml)


### Custom mosquitto.cnf configuration

The mosquitto image allows you to provide a custom `mosquitto.cnf` file for configuring mosquitto.
This Chart uses the `config` value to mount a custom `mosquitto.cnf` using a [ConfigMap](http://kubernetes.io/docs/user-guide/configmap/).
You can configure this by creating a YAML file that defines the `config` property as a multi-line string in the format of a `mosquitto.cnf` file.
For example:

```sh
cat > mosquitto-values.yaml <<EOF
config: |-
  log_dest stdout
  listener 1883
  listener 9090 
  protocol websockets
EOF

helm install --name mosquitto -f mosquitto-values.yaml queiroz/mosquitto
```

## Persistence

The [mosquitto](https://hub.docker.com/_/eclipse-mosquitto) image stores the mosquitto data at the `/mosquitto/data` path of the container.

The chart mounts a [Persistent Volume](https://kubernetes.io/docs/user-guide/persistent-volumes/) volume at this location. The volume is created using dynamic volume provisioning.
