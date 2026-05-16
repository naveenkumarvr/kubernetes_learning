# Setup Local Kind Cluster

This guide explains how to create a local Kubernetes cluster with Kind for learning and testing.

## Prerequisites

Make sure the following tools are available on your machine:

- Docker Desktop on macOS/Windows, or Docker Engine on Linux
- Kind
- kubectl

If you do not have them installed yet, use the official setup pages below:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Docker Engine for Linux: https://docs.docker.com/engine/install/
- Kind: https://kind.sigs.k8s.io/docs/user/quick-start/
- kubectl: https://kubernetes.io/docs/tasks/tools/

You can verify the installation with:

```bash
docker --version
kind --version
kubectl version --client
```

## Cluster Configuration

This setup uses the local [kind-config.yaml](kind-config.yaml) file.

That file defines the Kind cluster layout, including:

- one control-plane node
- two worker nodes
- port mappings for HTTP on `80` and HTTPS on `443`
- API server exposure settings for local access

Each line in [kind-config.yaml](/Users/naveen/Desktop/naveen-data/code/temp/kubernetes_learning/setup-local-cluster/kind-config.yaml) has already been explained with inline comments so it is easier to understand why each setting is used.

## Create The Cluster

Run the following command from this directory:

```bash
kind create cluster --config kind-config.yaml --name demo-lab01
```

## Verify The Cluster

After the cluster is created, confirm that it is running:

```bash
kubectl cluster-info
kubectl get nodes
```

You should see one control-plane node and two worker nodes in the output.

