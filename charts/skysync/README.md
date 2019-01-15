# SkySync

[SkySync](https://www.skysync.com/) is a file sync and migration solution used by many enterprises to securely migrate, copy, and synchronize files across storage system. Whether you’re ready to bridge your disconnected storage environment or looking to integrate and deploy new on-premises systems and/or cloud-based EFSS services, [SkySync](https://www.skysync.com/) is here.

## Introduction

This chart bootstraps a SkySync deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.6+ with Beta APIs enabled
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `migrator`:

```bash
$ helm install --name migrator skysync/skysync
```

The command deploys SkySync on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

By default a random password will be generated for the root user. If you'd like to set your own password change the skysyncPassword
in the values.yaml.

You can retrieve your root password by running the following command. Make sure to replace [YOUR_RELEASE_NAME]:

    printf $(printf '\%o' `kubectl get secret [YOUR_RELEASE_NAME]-skysync -o jsonpath="{.data.skysync-password[*]}"`)

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `migrator` deployment:

```bash
$ helm delete migrator
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the SkySync chart and their default values.

| Parameter                            | Description                               | Default                                              |
| ------------------------------------ | ----------------------------------------- | ---------------------------------------------------- |
| `imagePullPolicy`                    | Image pull policy                         | `IfNotPresent`                                       |
| `skysyncLicense`                     | License for the SkySync nodes.            | `nil` |
| `skysyncPassword`                    | Password for the `admin` user.            | Random 20 characters |
| `mssql-linux.sapassword`             | SA Password for the underlying SQL Server database.            | `this!is!not!a!secure!default!123` |
| `nodeSelector`                       | Node labels for pod assignment            | {}                                                   |
| `ssl.key`                            | TLS/SSL Server key (private key)          | `nil`                                                |
| `ssl.password`                       | Password for provided TLS/SSL Server key  | `nil`                                                |

Some of the parameters above map to the env variables defined in the [SkySync DockerHub image](https://hub.docker.com/_/skysync/).

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install --name migrator \
  --set skysyncPassword=Sky*Sync2018! \
    skysync/skysync
```

The above command sets the SkySync `admin` account password to `Sky*Sync2018!`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install --name migrator -f values.yaml skysync/skysync
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## SSL

This chart supports configuring SkySync to use HTTPS with TLS/SSL certificates provided by the user. This is accomplished by storing the required server private key as a Kubernetes secret. The SSL options for this chart support the following use cases:

* Manage certificate secrets with helm
* Manage certificate secrets outside of helm

## Manage certificate secrets with helm

Include your certificate data in the `ssl.certificates` section. For example:

```
ssl:
  enabled: false
  secret: skysync-ssl-certs
  certificates:
  - name: skysync-ssl-certs
    key: |-
      -----BEGIN RSA PRIVATE KEY-----
      ...
      -----END RSA PRIVATE KEY-----
```

> **Note**: Make sure your certificate data has the correct formatting in the values file.

## Manage certificate secrets outside of helm

1. Ensure the certificate secret exist before installation of this chart.
2. Set the name of the certificate secret in `ssl.secret`.
3. Make sure there are no entries underneath `ssl.certificates`.

To manually create the certificate secret from local files you can execute:
```
kubectl create secret generic skysync-ssl-certs \
  --from-file=server-key.pem=./ssl/server-private-key.pem
```
> **Note**: `server-key.pem` **must** be used as the key name in this generic secret.
