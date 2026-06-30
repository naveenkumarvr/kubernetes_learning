# Dive into SPIFFE IDs

Diving into SPIFFE IDs is the perfect way to understand how real-world zero-trust production networks operate! 🌐 Instead of relying on volatile parameters like IP addresses, systems use a standardized, cryptographically bound naming convention.

A SPIFFE ID is essentially a structured URL that acts as a global, uniform passport for a workload. Let's dissect the exact anatomy of the ID that Cilium checks during a mutual authentication (mTLS) handshake:

```text
spiffe://   cluster.local   /ns/   app-space   /sa/   default
  └───┬───┘   └─────┬─────┘          └───┬───┘          └───┬───┘
   Scheme     Trust Domain           Namespace       Service Account
```

## 🪪 The Anatomy of a Workload Identity

Let's break down what each component tells the Envoy proxy:

- Scheme (spiffe://): Specifies the standard protocol format being used. 📜

- Trust Domain (cluster.local): Defines the boundary of trust. Only workloads within this same domain (or explicitly federated domains) can trust each other's passports. 🌍

- Namespace (app-space): The exact logical boundary in Kubernetes where the pod is running. This prevents a compromised pod in a dev namespace from claiming it belongs in production. 🏗️

- Service Account (default): The granular identity template attached to the pod's execution context. 🔑