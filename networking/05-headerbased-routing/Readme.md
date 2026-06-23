# Header-Based Routing
In this section we will learn how to control the traffic and redirect to different service based on HTTP header 
## Prerequisites

Make sure the following tools are available on your machine:

- Set up your home lab cluster using kind: [setup-local-cluster](../../setup-local-cluster/Readme.md)
- Install Cilium: [cilium setup](../cilium/Readme.md)
- Deploy and test [Gateway API](../01-gateway-api-cilium/readme.md)

## Practicals

### 1. Deploying Workload

Create the namespace and deploy both application versions:

```bash
# Create namespace
kubectl create ns app-space

# Deploy app version 1 with service
kubectl apply -f app-v1.yaml -n app-space

# Deploy app version 2 with service
kubectl apply -f app-v2.yaml -n app-space
```

### 2. Applying Header-Based HTTPRoute

Create an HTTPRoute that redirects traffic based on the `X-Beta-User` header:
- Requests with `X-Beta-User: true` go to app version 2
- All other requests go to app version 1

```bash
kubectl apply -f headerbased-httproute.yaml
```

## Testing

Test the header-based routing with and without the beta user header:

```bash
# Test without header (should route to Version 1)
curl -i http://172.19.0.2/

# Test with header (should route to Version 2)
curl -s -i -H "X-Beta-User: true" http://172.19.0.2/
```