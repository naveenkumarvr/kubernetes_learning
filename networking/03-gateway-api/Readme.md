# Gateway API Practice (Kind + Cilium)

This guide helps you practice Kubernetes Gateway API on a local Kind cluster using Cilium as the Gateway controller.

## Prerequisites

Before starting, make sure these are ready:

- Local Kind cluster setup: [setup-local-cluster](../../setup-local-cluster/Readme.md)
- Cilium installed with Gateway API enabled: [Cilium setup](../cilium/Readme.md)
- Tools available: `kubectl`, `kind`, and `cloud-provider-kind`

You can quickly verify Gateway API support:

```bash
kubectl get crd gateways.gateway.networking.k8s.io
kubectl get gatewayclass
```

Expected: a `cilium` `GatewayClass` should exist.

## Practice Flow

### 1) Create namespace and deploy Gateway

```bash
cd networking/03-gateway-api

# Create namespace for shared infra components
kubectl create namespace infra-gateway

# Deploy Gateway
kubectl apply -f gateway.yaml
```

Verify:

```bash
kubectl get gateway -n infra-gateway
kubectl describe gateway infra-gateway -n infra-gateway
```

Check these conditions in output:

- `Accepted: True`
- `Programmed: True`

### 2) Deploy sample apps in separate namespaces

```bash
kubectl create namespace site-a
kubectl create namespace site-b

# Deploy echo app in site-a
kubectl run web-a -n site-a --image=hashicorp/http-echo --port=5678 --expose -- -text="Welcome to Site A"

# Deploy echo app in site-b
kubectl run web-b -n site-b --image=hashicorp/http-echo --port=5678 --expose -- -text="Welcome to Site B"
```

Verify:

```bash
kubectl get pods,svc -n site-a
kubectl get pods,svc -n site-b
```

### 3) Ensure external IP assignment on Kind

Kind does not provide external IPs by default for `LoadBalancer` behavior.

Use one of these options:

- Recommended: run `cloud-provider-kind`
- Alternative: port-forward to the Gateway service created by Cilium

Run in a separate terminal and keep it running:

```bash
sudo cloud-provider-kind
```

Then re-check Gateway address:

```bash
kubectl get gateway -n infra-gateway
```

### 4) Apply HTTPRoutes

This lab uses path-based routing:

- `/weba` -> `site-a/web-a`
- `/webb` -> `site-b/web-b`

Apply routes:

```bash
kubectl apply -f http-route-site-a.yaml
kubectl apply -f http-route-site-b.yaml
```

Verify routes:

```bash
kubectl get httproute -n site-a
kubectl get httproute -n site-b

kubectl describe httproute web-a-route -n site-a
kubectl describe httproute web-b-route -n site-b
```

Check that route conditions include `Accepted: True`.

## Test

```bash
kubectl get gateway infra-gateway -n infra-gateway

export GATEWAY_IP=$(kubectl get gateway infra-gateway -n infra-gateway -o jsonpath='{.status.addresses[0].value}')

curl -H "Host: claybrainer.lab" http://$GATEWAY_IP/weba
curl -H "Host: claybrainer.lab" http://$GATEWAY_IP/webb
```

Expected response:

- `/weba` returns `Welcome to Site A`
- `/webb` returns `Welcome to Site B`

## Common Issues

### Gateway has no address

- Confirm `cloud-provider-kind` is running.
- Wait 15-30 seconds and check again:

```bash
kubectl get gateway -n infra-gateway -w
```

### HTTPRoute not accepted

- Ensure Gateway exists in `infra-gateway`.
- Ensure `parentRefs.name` is `infra-gateway` and `parentRefs.namespace` is `infra-gateway`.
- Confirm listener allows routes from all namespaces (`from: All` in `gateway.yaml`).

### Curl fails or times out

- Verify workloads and services:

```bash
kubectl get pods,svc -n site-a
kubectl get pods,svc -n site-b
```

- Verify route and gateway status with `kubectl describe`.

## Cleanup

```bash
kubectl delete namespace site-a site-b infra-gateway
```