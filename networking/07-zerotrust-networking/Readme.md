# ZeroTrust Networking

Welcome to Zero Trust Networking 🔐! In standard Kubernetes clusters, if a pod knows another pod's IP address, it can generally send traffic to it. Zero trust turns this upside down: never trust, always verify. Even if a packet comes from a valid pod IP, Cilium will block it unless the pod proves its cryptographic identity.

With Cilium, this is achieved by combining two core pillars:

- Cryptographic Identity (SPIFFE/SPIRE): Every pod is issued a unique, verifiable certificate (like a digital passport 🪪).
- Mutual Authentication (mTLS): Cilium automatically encrypts and authenticates the transport channel between pods using these digital passports, completely handling the handshake in the data path without touching your application code.

## Cilium Security Policy

Normally, a network policy blocks or allows traffic based on labels or IP addresses. With Cilium's Mutual Authentication feature, when pod A tries to talk to pod B: 

- The eBPF data path catches the network packet. 🔌 
- It checks the policy and sees that mutualAuthentication is required. 🔐
- Cilium handles an internal mTLS handshake using the SPIFFE IDs embedded in the certificates. 🪪
- If the certificates match and are valid, the traffic is allowed to flow. 🟢

### Step 1: ZeroTrustPolicy Creation

#### Setup

```bash
# Create the apps
kubectl apply -f app-v1.yaml -n app-space
kubectl apply -f app-v2.yaml -n app-space

# Create Httproute (Here we are using Header based routing)
kubectl apply -f http-route.yaml -n app-space

# Now Apply our policy which mandates authentication for v2 service request
kubectl apply -f zerotrust-policy.yaml -n app-space
```

#### Testing

```bash
# Testing normal request
curl -i http://172.19.0.2/
## Expected output 200

# Testing to V2 service
curl -i -H "X-Beta-User: true" http://172.19.0.2/
## Expected Output HTTP/1.1 503 Service Unavailable
```

That HTTP 503 Service Unavailable is Cilium actively enforcing Zero Trust. Because our policy mandates mutual authentication (mode: "required"), the Envoy proxy at the gateway tries to establish a secure mTLS handshake with app-v2. However, since the pods do not yet have valid SPIFFE/SPIRE cryptographic certificates (their digital passports), the handshake fails completely, and Cilium drops the connection to safeguard the backend.

Let's look under the hood to see how Cilium is tracking this identity wall. Run this command to check the secure identities Cilium has assigned to the pods in your namespace:

```bash
kubectl get ciliumidentities

kubectl describe ciliumidentities
```

When you look at the output, do you see distinct identity numbers assigned to app-v1 and app-v2

When you run kubectl get ciliumidentities, you will see a list of security identities that Cilium automatically assigns to pods based on their labels. However, these standard Cilium identities are metadata-driven; they don't include the actual cryptographic SPIFFE certificates needed for an mTLS network handshake.

To bridge this gap and get rid of that 503 Service Unavailable error, we have to issue those certificates. In a standard production setup with Cilium, this is handled by installing SPIRE (the SPIFFE Runtime Environment) in the cluster. Once SPIRE is active, it automatically injects a secure sidecar or certificate agent that hands out a unique, cryptographically verifiable identity to your workloads.

#### 🔒 How Cilium Validates It

When your header-based routing directed traffic to app-v2, Cilium's underlying data path intercepted the connection. It checked the cryptographic certificate presented by the sender to see if the embedded SPIFFE ID matched the allowed criteria in your CiliumNetworkPolicy. Because no certificate was found matching this strict format, Cilium threw the 503 Service Unavailable error to protect the system.