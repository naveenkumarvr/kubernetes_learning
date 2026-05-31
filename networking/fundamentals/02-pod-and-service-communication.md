# Kubernetes Networking Practical: Pod and Service Communication

This practical explains how Kubernetes networking works by testing:

- Kubernetes Services
- Stable service communication
- EndpointSlices
- Behavior when pods restart

By the end, you will understand how Services provide stable access even when pod IPs change.

If you want to learn the concept in detail, refer to my blog: [Read the blog here](https://claybrainer.com/kubernetes-networking-part-1-the-fundamentals).

## Prerequisites

Make sure you have:

- A running Kubernetes cluster (Minikube, Kind, EKS, AKS, GKE, K3s, etc.)
- `kubectl` configured

Verify your cluster:

```bash
kubectl get nodes
```

You should see nodes in the `Ready` state.

If you need to set one up locally, follow [Setup Local Kind Cluster](../../setup-local-cluster/Readme.md).

## Step 1: Deploy a Simple Application

Deploy an NGINX application with three replicas.

Create a file called `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Apply it:

```bash
kubectl apply -f deployment.yaml
```

Check pods:

```bash
kubectl get pods -o wide
```

Notice:

- Every pod has a unique IP
- Pods may run on different nodes

## Step 2: Create a Kubernetes Service

Now create a Service so clients can use a stable endpoint.

Create a file called `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply the Service:

```bash
kubectl apply -f service.yaml
```

Check Service details:

```bash
kubectl get svc nginx-service
```

The Service receives a stable `CLUSTER-IP` that does not change when pods restart.

## Step 3: Test Service Communication

Start a temporary test pod:

```bash
kubectl run network-test --image=curlimages/curl -it --rm -- sh
```

Use Service DNS:

```sh
curl http://nginx-service
```

Use Service ClusterIP:

```sh
curl http://<SERVICE_CLUSTER_IP>
```

You should get the NGINX page in both cases.

Traffic now follows:

```text
Pod -> Service -> Pod
```

Instead of:

```text
Pod -> Pod IP
```

## Step 4: Inspect EndpointSlices

List EndpointSlices:

```bash
kubectl get endpointslices
```

Describe the EndpointSlice for `nginx-service`:

```bash
kubectl describe endpointslice <ENDPOINTSLICE_NAME>
```

You will see pod IPs for healthy backends. This is how Kubernetes knows where to route Service traffic.

## Step 5: Test Pod Failure and Recovery

List pods:

```bash
kubectl get pods
```

Delete one NGINX pod:

```bash
kubectl delete pod <POD_NAME>
```

Watch pod recreation:

```bash
kubectl get pods -w
```

The replacement pod gets a new IP. Even so, requests to `nginx-service` continue to work because Kubernetes updates EndpointSlices automatically.

## Cleanup

Delete created resources:

```bash
kubectl delete deployment nginx-demo
kubectl delete service nginx-service
```

## What You Learned

- Every pod gets a unique IP
- Services provide stable networking
- EndpointSlices track healthy pod backends
- Kubernetes automatically handles pod recreation and routing updates
