# Cilium FQDN based filtering

There are scenarios where we want to communicate with the exteranl APIs such as api.stripe.com or api.paytm.com. In this cases we cannot track the ip address of the external API and add it to the allowed list as they use Load balancer and CDN and their IP tends to change. Tracking and update CIDR is a tideous job. Trusting it blindly allow all traffic is also a problem. 

This is where Ciliun FQDN based whiltlisting comes in to play. 

## How it works ?

Cilium uses **DNS Snooping**. When a pod assume frontend pod makes a DNS request for api.github.com Cilium intercepts the DNS **Response** reads the dynamically assigned IP addresses and instantly updates the internal eBPF kernel maps to allow outgoing traffic to those specific IP. 

Since entire mechanism relies on reading the pods DNS conversation, if a hardcoded IP is added to the pod and try to reach the github.com will not work as the ip address is not added to the pod. 

## Example

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: frontend-egress-fqdn
  namespace: policy-lab
spec:
  endpointSelector:
    matchLabels:
      app: frontend
    egress:
    - toFQDNs:
      - matchName: "api.github.com"
      toPorts:
      - ports:
        - port: "443"
          protocol: TCP
```

## Practicals.

- Deploy the apps follow [Readme.md](../app/Readme.md)
- Apply the manifest file using this command: `kubectl apply -f fqdn-whitelisting.yaml -n policy-lab`

## Testing

- Testing allowed list

```bash
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -I https://api.github.com

# Expected Output: 200
```

- Failure Case

```bash
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -I https://google.com

# Expected output :
```