# Namespace Isolation

In a normal Kubernetes scenario, all namespaces in a cluster can communicate with other namespaces. To enhance security, we can block cross-namespace communication.

This can be implemented in two ways on Cilium:

### Namespaced Policies (CiliumNetworkPolicy)
- **Scope**: Bound to a single, specific namespace.
- **Persona**: Used by Application Teams.
- **Use Case**: A developer in tenant-prod wants to say: "Only allow traffic into my specific service if it comes from within my own namespace."

### ClusterWide Policies (CiliumClusterwideNetworkPolicy)
- **Scope**: Applies globally across the entire Kubernetes cluster, regardless of namespaces.
- **Persona**: Enforced by Platform & Security Infrastructure Teams.
- **Use Case**: Setting a global governance rule: "No namespace in the entire cluster is allowed to talk to any other namespace unless explicitly whitelisted."

## How the Policy Logic Works

When you apply a Default-Deny policy to tenant-prod, Cilium programs the Linux kernel (via eBPF):

1. Every time a network packet leaves a pod in tenant-dev bound for an IP address in tenant-prod, the kernel intercepts it.
2. It checks the packet's source identity metadata.
3. If the source namespace label (`io.kubernetes.pod.namespace`) does not match the allowed rules, the packet is immediately dropped at the source/ingress virtual interface, preventing it from ever reaching the target container's application loop.

# Practicals

## 1. Initialize the Multi-Tenant Environments

```bash
# 1. Create the tenant namespaces
kubectl create namespace tenant-prod
kubectl create namespace tenant-dev

# 2. Spin up a target web service in the production namespace
kubectl run prod-web -n tenant-prod --image=hashicorp/http-echo -- -text="Welcome to Production Environment"

# 3. Create a ClusterIP Service to expose the production web server
kubectl expose pod prod-web -n tenant-prod --port=80 --target-port=5678

# 4. Spin up an interactive testing client in the development namespace
kubectl run dev-client --image=nicolaka/netshoot -n tenant-dev -- sleep 3600
```

## 2. Establish the Baseline Test (The Security Breach)

Before applying any rules, let's prove that the cluster is completely open. We will execute a curl command from the dev-client pod straight into the prod-web service.

```bash
kubectl exec -n tenant-dev dev-client -- curl -i --connect-timeout 5 http://prod-web.tenant-prod.svc.cluster.local
```

## 3. Apply the Iron Curtain (The Isolation Policy)

```bash
kubectl apply -f namespace-isolation.yaml
```

### Test

```bash
kubectl exec -n tenant-dev dev-client -- curl -i --connect-timeout 5 http://prod-web.tenant-prod.svc.cluster.local
```

### Expected Output

```
curl: (28) Connection timed out after 5001 milliseconds
command terminated with exit code 28
```

**Cilium's Invisible Wall is Active**: Instead of letting the network packet pass into the application, Cilium's eBPF program intercepted the packet at the Linux kernel level. It analyzed the packet's metadata, saw that it originated from the namespace tenant-dev, recognized that our CiliumNetworkPolicy explicitly forbids cross-namespace ingress into tenant-prod, and silently dropped the packet into a black hole.

## 4. Building a Controlled Exception

In a real enterprise, completely blocking namespaces from talking to each other is only half the battle. Eventually, a real business case arrives: The engineering team needs a microservice inside tenant-dev to securely consume a database or a single specific API inside tenant-prod.

Let's learn how to punch a highly targeted, secure hole through our namespace wall without tearing down the entire security policy.

We will write an updated policy that allows traffic into tenant-prod ONLY if it matches two conditions:

1. It originates from the tenant-dev namespace.
2. AND the pod explicitly carries the label `app: authorized-client`.

Any other pod in the dev namespace (like a standard compromised container or debugging utility) will still be met with a silent timeout.

```bash
kubectl label pod dev-client -n tenant-dev app=authorized-client

kubectl apply -f selective-isolation.yaml

# Test
kubectl exec -n tenant-dev dev-client -- curl -i --connect-timeout 5 http://prod-web.tenant-prod.svc.cluster.local
```

### Expected Output

```
HTTP/1.1 200 OK
...
```

### What Happened Under the Hood?

Let's trace exactly how Cilium processed that connection:

1. When you added the label `app=authorized-client` to the dev-client pod, Cilium's local agent instantly recognized the change and assigned it a new numerical security identity.
2. When the packet arrived at prod-web, Cilium's eBPF program intercepted it at the virtual interface.
3. Instead of dropping it, the eBPF program evaluated Rule B in your new CiliumNetworkPolicy. It verified that the sender's metadata matched both the source namespace (tenant-dev) AND the required application identity label (authorized-client).
4. Because it was a perfect match, the packet was safely forwarded directly into the application container.

## Advanced: Using Dynamic Namespace Selectors

### How Cilium Evaluates `"k8s:io.kubernetes.pod.namespace": "from-endpoint"` Dynamically

When a packet tries to go from tenant-dev to tenant-dev, Cilium checks: Does the source namespace (tenant-dev) match the destination namespace (tenant-dev)? ✅ **Yes! Traffic allowed.**

When your packet tries to go from tenant-dev to tenant-prod, Cilium checks: Does the source namespace (tenant-dev) match the destination namespace (tenant-prod)? ❌ **No! Traffic dropped.**

### Why Platform Engineers Use This Pattern

If you type `tenant-dev`, that policy only works for that specific namespace. If your company creates 100 new namespaces tomorrow (tenant-team-a, tenant-team-b, etc.), you would have to write 100 different rules!

By using `"k8s:io.kubernetes.pod.namespace": "from-endpoint"`, you write one single policy at the cluster level, and it automatically enforces a strict namespace isolation bubble around every single namespace in the cluster simultaneously.