# rook

## Introduction

This document describes the process of deploying `rook` storage solutions on a kubernetes cluster, rook automates the deployment, management and self-healing of the following storage solutions:

* CEPH
* NFS
* Cassandra

In our case, we will use CEPH cluster.

To read more about different types of storage, please reffer to the following [document](./Documentation/Storage.md)

To read more about CEPH, please reffer to the following [document](./Documentation/ceph.md)

## Deployment

### Prerequisites

In order to deploy rook in an on-premise cluster, we need a cluster with at least 3 worker nodes.

Ceph composes the cluster from the disks plugged in each worker node.

Notice that these disks must not be formatted or have a filesystem

Kubernetes v1.16 or higher is supported by Rook.

### Installation 

We will be using the version `1.8.2` of rook.

Create the needed resources and the operator:

Notice that in an on promise clusters, we need to enable the `ROOK_ENABLE_DISCOVERY_DAEMON` parameter in order for rook to auto detect the attached disks on worker nodes. 

```
kubectl create -f manifests/operator/
```

Watch the status of the operator

```
watch kubectl get pods
```

After the operator, we need to deploy a ceph cluster

```
kubectl create -f manifests/cluster
```

After the deployment of the cluster, we can access the ceph dashboard to see the cluster status.

To access the cluster dashboard we will create a nodeport service.

```
kubectl create -f manifests/service
```

Now, access the URL of the dashboard:

```
https://IP_OF_WORKER_NODE:30007
```

The username is `admin` and the password can be retrieved from the secret `rook-ceph-dashboard-password`.

The deployed cluster offers 3 types of storage:

* Block: Create block storage to be consumed by a pod (RWO)
* Shared Filesystem: Create a filesystem to be shared across multiple pods (RWX)
* Object: Create an object store that is accessible inside or outside the Kubernetes cluster

In the next part, we will demonstrate how to deploy an application that consumes a shared filesystem in (RWX) mode.
## Deploying a demo application

Now that we have a ceph cluster up & running, we will deploy a demo application.

First of all, we need to deploy the filesystem:

```
kubectl create -f manifests/filesystem/filesystem.yaml
```

To confirm the filesystem is configured, wait for the mds pods to start

```
kubectl -n rook-ceph get pod -l app=rook-ceph-mds
```

Once the filesystem is created, we will create a storageclass to automate teh creation of persistent volumes by kubernetes based on the fileystsem we created earlier.

```
kubectl create -f manifests/storageclass/storageclass.yaml
```

Once the storage class is created, all we need to deploy is the deployment manifest and the persistent volume claim.

```
kubectl create -f manifests/demoapp/main.yaml
```