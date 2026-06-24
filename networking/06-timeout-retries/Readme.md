# Timeout and Retry (Gateway API)
In this section, we learn how to enforce request time limits using Gateway API HTTPRoute timeouts.

## Prerequisites

Make sure the following are ready:

- Set up your local cluster: [setup-local-cluster](../../setup-local-cluster/Readme.md)
- Install Cilium: [cilium setup](../cilium/Readme.md)
- Deploy and verify Gateway API: [gateway api setup](../01-gateway-api-cilium/readme.md)

## Practicals

### 1. Deploy workload

Create namespace and deploy app services:

```bash
# Create namespace (ignore error if it already exists)
kubectl create ns app-space

# Deploy version 1 and version 2 workloads
kubectl apply -f app-v1.yaml
kubectl apply -f app-v2.yaml
```

### 2. Apply timeout HTTPRoute

Apply the correct route manifest:

```bash
kubectl apply -f timeout-retries.yaml
```

This route currently configures:

- `request: 1s` -> full downstream request timeout
- `backendRequest: 250ms` -> timeout per upstream/backend attempt

## About retries

Standard Gateway API HTTPRoute `v1` does not support `retry` under `backendRefs`.
If you add retry fields there, Kubernetes returns a strict decoding error.

To use retries, configure them through an implementation-specific policy (for example, Cilium/Envoy-specific resources) instead of native HTTPRoute fields.

## Testing

### 1. Verify resources

```bash
kubectl get httproute -n app-space
kubectl describe httproute app-resilience-route -n app-space
```

### 2. Functional test (normal traffic)

Replace `<GATEWAY_IP>` with your Gateway/LoadBalancer IP.

```bash
curl -i --max-time 3 http://<GATEWAY_IP>/
```

Expected:

- `HTTP/1.1 200 OK`
- Response body from `svc-v1` ("Responding from Version 1")

### 3. Observe timeout behavior with a strict client timeout

This command intentionally sets a very low client timeout.

```bash
curl -i --max-time 0.1 http://<GATEWAY_IP>/
```

Expected:

- Curl may fail with client-side timeout because `--max-time` is too small.
- This confirms the client timeout boundary separately from Gateway timeout behavior.

### 4. Run repeated requests and watch for timeout responses

```bash
for i in {1..30}; do
	curl -s -o /dev/null -w "%{http_code}\n" http://<GATEWAY_IP>/
done
```

Expected:

- Mostly `200` for healthy backend responses.
- If backend is slow/unhealthy, you may see timeout-related status codes from the Gateway.

### 5. Quick troubleshooting

```bash
kubectl get pods -n app-space -o wide
kubectl get svc -n app-space
kubectl get events -n app-space --sort-by=.lastTimestamp | tail -n 20
```