# Pod-to-Pod Communication

In Kubernetes, pods can communicate directly over the cluster network without an intermediate network layer.

If you want to learn the concept in detail, refer to my blog: [Read the blog here](<BLOG_URL_PLACEHOLDER>).

This document is the practical companion to that blog and demonstrates the behavior with a simple hands-on test.

## Prerequisites

Make sure you already have a working Kubernetes cluster.

If you need to set one up locally, follow [Setup Local Kind Cluster](../../setup-local-cluster/Readme.md).

## Pod-to-Pod Communication Test

### Step 1: Create Two Test Pods

Create a file named `test-pods.yaml` and paste the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  labels:
    app: test
spec:
  containers:
    - name: nginx
      image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  labels:
    app: test
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
```

Apply the manifest:

```bash
kubectl apply -f test-pods.yaml
```

### Step 2: Get Pod IP Addresses

Run:

```bash
kubectl get pods -o wide
```

Note the IP addresses of `pod-a` and `pod-b`.

### Step 3: Open a Shell in pod-b

Run:

```bash
kubectl exec -it pod-b -- sh
```

This opens a shell session inside `pod-b`.

### Step 4: Access pod-a from pod-b

From inside `pod-b`, run:

```sh
wget -qO- http://<pod-a-ip>
```

If pod-to-pod networking is working, you should see the NGINX welcome page HTML.

## What This Proves

- Pod-to-pod communication works directly over cluster networking.
- A Service is not required for direct pod IP connectivity.
- Kubernetes provides a flat network model for pods.