# Kubernetes Native Network Policies

In this section we will see how to configure Kubernetes Native network policies and what are its adv and disadv

## How Native Kubernetes Network Policies Work
Before writing YAML, there are three fundamental rules you must understand about native Kubernetes network policies:

They are Whitelist-Only (Additive): You cannot explicitly write a "Deny" rule in native Kubernetes. You can only define what traffic is allowed.

The "Isolation" Flip-Switch: By default, pods are Non-Isolated; they accept traffic from anywhere. However, the moment a pod is selected by any NetworkPolicy (via podSelector), it instantly flips into an Isolated state. Any traffic not explicitly whitelisted in that policy is immediately dropped.

Independent Directions: Ingress (incoming) and Egress (outgoing) are treated separately. A policy can isolate a pod's incoming traffic while leaving its outgoing traffic wide open, or vice-versa.

It is incredibly common to wonder how network policies interact with Services (like http://backend). Here is the core rule: Kubernetes Network Policies completely ignore Services; they only care about Pods.

When your frontend pod runs curl http://backend, here is exactly what happens under the hood:

- **DNS Resolution**: The pod asks CoreDNS for the IP of backend. CoreDNS returns the **Service ClusterIP** (e.g., 10.96.x.x).

- **Service Translation**: As the packet leaves the container, **Cilium (using eBPF) instantly intercepts it and replaces the destination Service ClusterIP with the actual Pod IP** of one of your backend replicas (e.g., 10.244.x.x).

- **Policy Enforcement**: Only after the packet is addressed to the real Pod IP does the Network Policy check take place.

Because the Service layer vanishes before the firewall rules are evaluated, your Network Policies must always target pods via podSelector, even if your traffic goes through a Service.


## Practical Implementation

- Create Namespace `kubectl create ns policy-lab`
- Deploy Three tier application
    ```bash
    kubectl apply -f app/db.yaml -n policy-lab

    kubectl apply -f app/backend.yaml -n policy-lab

    kubectl apply -f app/frontend.yaml -n policy-lab
    ```
- Deploy the policy `kubectl apply -f k8-native-networkpolicy/allow-strict-path.yaml`

### Testing
```bash
# Frontend to Backend
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://backend

## Expected Ouput: 200


# Backend to DB
kubectl exec -n policy-lab deploy/backend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://database


## Expected Ouput: 200


# Frontend to Direct DB
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 http://database


## Expected Ouput: curl: (28) Connection timed out
```