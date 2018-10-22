# Cosmos Helm Chart

Cosmos is a storage backend for Cloudserver that lets you manage data stored on a filesystem and other storage platforms.

## Introduction

This chart bootstraps a Cosmos deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

Before installing this chart, you must either have a Zenko or a standalone Cloudserver instance running.

## Installing the Chart

1. Configure the cosmos backend as a Cloudserver location constraint.

```sh
$ cat locationConfig.json
{
    "us-east-2": { // This is the region on the rclone.remote configuration
        "type": "pfs",
        "objectId": "nfs-42",
        "legacyAwsBehavior": true,
        "details": {
            "bucketName": "nfs",
            "bucketMatch": true,
            "serverSideEncryption": true,
            "supportsVersioning": false,
            "mountPath": "/data", // This should match the PV path
            "pfsDaemonEndpoint": { // This should match the cosmos pfsd endpoint
                "host": "my-release-cosmos-pfsd",
                "port": "80"
            }
        }
    }
}
```

2. Create a PV backed by your desired storage plaform. For example, an NFS-backed PV:

```sh
$ cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 300Gi
  nfs:
    path: /data
    server: 10.100.1.42
  persistentVolumeReclaimPolicy: Retain
```

3. Create a PVC for the just created PV.

```sh
$ cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    helm.sh/resource-policy: keep
  name: cosmos
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 300Gi
  volumeName: nfs-pv
EOF
```

4. Configure the `rclone.remote` values in the `values.yaml` file. For example:

```yaml
rclone:
  remote:
    accessKey: my-access-key
    secretKey: my-secret-key
    endpoint: http://cloudserver.local
    region: us-east-2
```

> **Tip**: This can also be perfomed via command line arguments to the below `helm install` command.

5. Install the chart.

```bash
$ helm install --name my-release ./cosmos
```

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Prisma chart and their default values.

| Parameter              | Description                             | Default                      |
| ---------------------- | --------------------------------------- | ---------------------------- |
| `pfsd.name` | Name of the pfsd component | `pfsd` |
| `pfsd.replicaCount` | Number of pfsd replicas| `1` |
| `pfsd.image.repository` | Pfsd image repository  | `gguiulfo/pfsd` |
| `pfsd.image.tag` | Pfsd image tag | `0.1` |
| `pfsd.image.pullPolicy` | Pfsd image pull policy | `IfNotPresent` |
| `pfsd.service.type` | Pfsd service type | `ClusterIP` |
| `pfsd.service.port` | Pfsd service port | `80` |
| `pfsd.resources` | Pfsd resource requests and limits | `{}` |
| `pfsd.nodeSelector` | Node labels for Pfsd pod assignment | `{}` |
| `pfsd.tolerations` | Node taints to tolerate | `[[` |
| `pfsd.affinity` | Pfsd pod affinity | `{}` |
| `rclone.name` | Name of the rclone component | `rclone` |
| `rclone.image.repository` | rclone image repository | `gguiulfo/rclone` |
| `rclone.image.tag` | rclone image tag | `0.2` |
| `rclone.image.pullPolicy` | rclone image pull policy | `IfNotPresent` |
| `rclone.schedule` | rclone CronJob schedule | `*/10 * * * *` |
| `rclone.successfulJobsHistory` | rclone CronJob successful job history | `1` |
| `rclone.remote.accessKey` | Remote backendj access key | `my-access-key` |
| `rclone.remote.secretKey` | Remote backend secret key | `my-secret-key` |
| `rclone.remote.endpoint` | Remote endpoint | `http://cloudserver.local` |
| `rclone.remote.region` | Remote region | `us-east-1` |
| `rclone.resources` | rclone resource requests and limits | `{}` |
| `rclone.nodeSelector` | Node labels for rclone pod assignment | `{}` |
| `rclone.tolerations` | Node taints to tolerate | `[]` |
| `rclone.affinity` | rclone pod affinity | `{}` |
| `persistentVolume.enabled` | If true, enable persistentVolume | `true` |
| `persistentVolume.accessModes` | Persistent Volume access modes | `ReadWriteMany` |
| `persistentVolume.existingClaim` | Exsisting clame name | `""` |
| `persistentVolume.storageClass` | Persistent Volume storage class | `cosmos` |
| `persistentVolume.size` | Persistent Volume size | `1Gi` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install ./cosmos --name my-release \
    --set pfsd.replicaCount=3
```

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
$ helm install ./cosmos --name my-release -f values.yaml
```

> **Tip**: You can use the default [values.yaml](values.yaml)
