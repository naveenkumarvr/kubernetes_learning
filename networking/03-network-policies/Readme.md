# Kubernetes Native Network Policies

In this section we will see how to configure Kubernetes native network policies, along with their advantages and limitations.

## How Native Kubernetes Network Policies Work

Before writing YAML, there are three fundamental rules you must understand about native Kubernetes network policies:

1. Whitelist-only (additive): You cannot explicitly write a Deny rule in native Kubernetes. You can only define what traffic is allowed.
2. Isolation flip-switch: By default, pods are non-isolated and accept traffic from anywhere. The moment a pod is selected by any NetworkPolicy (via podSelector), it becomes isolated for the selected direction, and non-whitelisted traffic is dropped.
3. Independent directions: Ingress (incoming) and Egress (outgoing) are treated separately. A policy can isolate one direction while leaving the other open.

It is common to wonder how network policies interact with Services (for example, http://backend). The core rule is that Kubernetes NetworkPolicy evaluates pod-to-pod traffic, not Service objects directly.

When your frontend pod runs curl http://backend, this is what happens:

1. DNS resolution: The pod asks CoreDNS for backend and gets a Service ClusterIP (for example, 10.96.x.x).
2. Service translation: Cilium (eBPF) intercepts and translates Service IP to a real backend pod IP (for example, 10.244.x.x).
3. Policy enforcement: Policy is enforced against the final pod destination.

Because Service translation happens before policy evaluation, your policies should target pods with podSelector.


## Practical Implementation

- Create namespace:
  ```bash
  kubectl create ns policy-lab
  ```
- Deploy three-tier application:
  ```bash
  kubectl apply -f app/db.yaml -n policy-lab
  kubectl apply -f app/backend.yaml -n policy-lab
  kubectl apply -f app/frontend.yaml -n policy-lab
  ```
- Deploy the strict path policy:
  ```bash
  kubectl apply -f k8NativeNetworkpolicy/allowStrictPath.yaml -n policy-lab
  ```

Policy file reference:

- k8NativeNetworkpolicy/allowStrictPath.yaml
- k8NativeNetworkpolicy/Readme.md

### Testing

```bash
# Frontend to Backend
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://backend

# Backend to DB
kubectl exec -n policy-lab deploy/backend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://database

# Frontend to Direct DB
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 http://database
```

Expected output:

1. Frontend to Backend: 200
2. Backend to DB: 200
3. Frontend to DB direct: curl timeout

Quick policy behavior summary:

1. frontend -> backend:80 is allowed
2. backend -> database:80 is allowed
3. frontend -> database:80 is denied
4. DNS egress on UDP/TCP 53 is allowed for all three workloads


# Cilium Policy

Traditional Kubernetes networking and older CNIs rely on iptables inside the Linux kernel to enforce rules. As clusters grow to hundreds of pods, evaluating thousands of sequential iptables rules slows down network performance.

Cilium uses eBPF (Extended Berkeley Packet Filter). It bypasses iptables entirely, running sandboxed programs directly inside the Linux kernel dynamically. This allows policy lookups to happen at near-wire speed regardless of how many rules or pods you have.

## Quick Comparison

| Feature | K8s Native | Calico | Cilium |
|---|---|---|---|
| Core technology | iptables / IPVS | iptables (eBPF optional) | eBPF native |
| Layer 7 filtering | No | Yes (requires Envoy) | Yes (built-in Envoy integration) |
| DNS/FQDN rules | No | Yes | Yes |
| Observability | Basic logging | Advanced features vary by edition | Hubble (built-in ecosystem) |

## Key Advantages Over Native Policies

1. Layer 7 awareness: Native policies only evaluate L3/L4 attributes (IP, protocol, port). Cilium can enforce HTTP-aware rules such as allowing only specific paths or methods.
2. FQDN-based control: Instead of managing changing external IPs, Cilium can enforce policies based on domain names (for example, api.stripe.com).

## Layer 7 (Application Layer) Policy: Application-Aware Security

Traditional Kubernetes policies stop at Layer 4 (TCP/UDP ports). If port 80 is open, any HTTP request can pass. Cilium can inspect Layer 7 details such as method and path.

Example scenario:

1. GET /api/v1/public (safe user endpoint)
2. DELETE /api/v1/admin (destructive admin endpoint)

With standard Kubernetes policy, if frontend is allowed to backend:80, both endpoints are reachable. With Cilium L7 policy, you can allow GET /api/v1/public while blocking DELETE /api/v1/admin on the same port. This is an implementation of the Principle of Least Privilege.