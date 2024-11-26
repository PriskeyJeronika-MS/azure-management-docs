---
title: Install Container Storage on a single-node Ubuntu cluster
description: Learn how to quickly install Container Storage on a single-node Ubuntu cluster.
author: sethmanheim
ms.author: sethm
ms.topic: quickstart
ms.date: 11/11/2024


---
  
# Quickstart: Install Azure Container Storage on a single-node Ubuntu cluster

This quickstart shows you how to install Azure Container Storage enabled by Azure Arc on a fresh single-node Ubuntu cluster.

## Prerequisites

Before you begin, you need the following prerequisites:

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free) before you begin.
- An Arc-connected Kubernetes cluster. To connect an existing Kubernetes cluster to Azure Arc, see [Connect an existing Kubernetes cluster to Azure Arc](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli).

### Parameters

You need the following parameter values to complete this quickstart:

| Parameter name  | Description                                                   |
|-----------------|---------------------------------------------------------------|
| `resource-group`  | The name of the Azure Resource Group that your cluster is in.  |
| `cluster-name`    | The name of your Arc-connected Kubernetes cluster.             |

## Step 1: Install Open Service Mesh (OSM)

The first step is to install Open Service Mesh (OSM) using the following command:

```azurecli
az k8s-extension create --resource-group "YOUR_RESOURCE_GROUP_NAME" --cluster-name "YOUR_CLUSTER_NAME" --cluster-type connectedClusters --extension-type Microsoft.openservicemesh --scope cluster --name osm \ 
--config "osm.osm.featureFlags.enableWASMStats=false" \ 
--config "osm.osm.enablePermissiveTrafficPolicy=false" \ 
--config "osm.osm.configResyncInterval=10s" \ 
--config "osm.osm.osmController.resource.requests.cpu=100m" \ 
--config "osm.osm.osmBootstrap.resource.requests.cpu=100m" \ 
--config "osm.osm.injector.resource.requests.cpu=100m"
```

## Step 2: Set maximum user instances

To determine if you set `fs.inotify.max_user_instances` to 1024, run the following command:

```bash
sysctl fs.inotify.max_user_instances
```

After you run this command, if it returns less than 1024, run the following command to increase the maximum number of files and reload the `sysctl` settings:

```bash
echo 'fs.inotify.max_user_instances = 1024' | sudo tee -a /etc/sysctl.conf 
sudo sysctl -p
```

## Step 3: Install Azure IoT Operations dependencies

Run the following command to install the Azure IoT Operations dependencies:

```azurecli
az k8s-extension create --cluster-name "${YOUR-CLUSTER-NAME}" --name "aio-certmgr" --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-type connectedClusters --extension-type microsoft.iotoperations.platform --scope cluster --release-namespace cert-manager
```

## Step 4: Install the Azure Container Storage enabled by Azure Arc extension

Install the Azure Container Storage enabled by Azure Arc extension using the following command:

```azurecli
az k8s-extension create --resource-group "${YOUR-RESOURCE-GROUP}" --cluster-name "${YOUR-CLUSTER-NAME}" --cluster-type connectedClusters --name azure-arc-containerstorage --extension-type microsoft.arc.containerstorage
```

> [!NOTE]
> By default, the `--release-namespace` parameter is set to `azure-arc-containerstorage`. If you want to override this setting, add the `--release-namespace` flag to the previous command and populate it with your details. Any values set at installation time persist throughout the installation lifetime (including manual and auto-upgrades).

> [!IMPORTANT]
> If you use OneLake, you must use a unique extension name for the `--name` parameter in the `az k8s-extension create` command.

## Configuration CRD

First, create a file named **edgeConfig.yaml** with the following contents:

```yaml
apiVersion: arccontainerstorage.azure.net/v1 
kind: EdgeStorageConfiguration 
metadata: 
  name: edge-storage-configuration 
spec: 
  defaultDiskStorageClasses: 
    - "default" 
    - "local-path" 
  serviceMesh: "osm"
```

To apply this .yaml file, run:

```bash
kubectl apply -f "edgeConfig.yaml"
```

## Next steps

Now that you have the extension installed, you can configure some volumes, either [Local Shared Edge Volumes](local-shared-edge-volumes.md) or [Cloud Ingest Edge Volumes](cloud-ingest-edge-volume-configuration.md).
