# Layer 7 Application-Aware Security
How Cilium Does It ? To achieve this, Cilium integrates an embedded Envoy proxy 🔄. When a Layer 7 policy is applied:

- eBPF hooks intercept the packet at the Linux kernel level.

- The packet is redirected to the internal Envoy proxy container sidecar or node-agent.

- Envoy inspects the actual HTTP headers (checking for GET vs DELETE and the URL path) and decides whether to forward or drop the traffic.

## Practicals
Since the traffic is moving from the frontend to the backend, it arrives as incoming traffic at the backend pod 🖥️. To block the unauthorized requests right as they try to enter the backend, we configure an ingress policy targeting the backend pod 🛡️.

```yaml
# cilium-l7-policy.yaml

apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: backend-l7-policy
  namespace: policy-lab
spec:
  endpointSelector:
    matchLabels:
      app: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/public"
```

```bash
kubectl apply -f cilium-l7-policy.yaml
```

### Testing
Our network-multitool web server handles any URL path you throw at it. This makes it perfect for testing how Cilium intercepts the traffic before it reaches the application.

- Let's run the allowed GET request first: 
    ```bash
    kubectl exec -n policy-lab deploy/frontend -- curl -s -o /dev/null -w "%{http_code}\n" http://backend/api/v1/public

    # Expected Output : 200
    ```
- Now, let's try the unauthorized DELETE request to the admin path:

    ```bash
    kubectl exec -n policy-lab deploy/frontend -- curl -X DELETE -s -o /dev/null -w "%{http_code}\n" http://backend/api/v1/admin

    # Expected Output : HTTP 403 Forbidden
    ```
- When a standard Layer 4 policy blocks traffic, it silently drops the packets, causing the connection to time out (returning a 000 status code). Because this Layer 7 policy uses the Envoy proxy, Cilium actively sends an HTTP error response back to the frontend container immediately.