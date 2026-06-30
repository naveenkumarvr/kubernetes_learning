# kubernetes_learning

This repository helps you learn Kubernetes by doing.

The repository is structured so that each component has its own section, where you can learn the concept in detail and follow practical exercises. Each section includes its own README with the necessary explanations, steps, and commands, so you can follow along and practice as you learn.

## Getting Started

To set up your local lab for learning Kubernetes, start with the [setup-local-cluster](setup-local-cluster/Readme.md) section.

That section explains how to create a local Kind-based Kubernetes cluster and prepares the environment used by the examples in this repository.

After your cluster is ready, configure Cilium using [networking/cilium/Readme.md](networking/cilium/Readme.md).

## Repository Topics

### Learning Guide

- [Learning Guide Overview](learning_guide/Readme.md)
- [Detailed Course Roadmap](learning_guide/DetaieldCourse.md)

### Networking Track

- [Networking Practicals Blueprint](networking/practicals.md)
- [Cilium Setup and Concepts](networking/cilium/Readme.md)
- [Gateway API with Cilium](networking/01-gateway-api-cilium/readme.md)
- [Gateway API TLS](networking/02-gateway-api-tls/Readme.md)
- [Network Policies Overview](networking/03-network-policies/Readme.md)
- [Kubernetes Native NetworkPolicy Lab](networking/03-network-policies/01-k8NativeNetworkpolicy/Readme.md)
- [Cilium Layer 7 Policy Lab](networking/03-network-policies/02-layer7PolicySecurityCilium/Readme.md)
- [FQDN Whitelisting Lab](networking/03-network-policies/03-fqdn-whitelisting/Readme.md)
- [Three-Tier App Manifests](networking/03-network-policies/app/Readme.md)
- [Traffic Splitting (Canary) Lab](networking/04-traffic-splitting/Readme.md)
- [Header-Based Routing Lab](networking/05-headerbased-routing/Readme.md)
- [Timeout and Retries Lab](networking/06-timeout-retries/Readme.md)
- [Zero Trust Networking Lab](networking/07-zerotrust-networking/Readme.md)
- [SPIFFE Identity Deep Dive](networking/07-zerotrust-networking/SPIDEE.md)

## Suggested Learning Order

1. [Setup Local Cluster](setup-local-cluster/Readme.md)
2. [Cilium Setup](networking/cilium/Readme.md)
3. [Gateway API Basics](networking/01-gateway-api-cilium/readme.md)
4. [Gateway API TLS](networking/02-gateway-api-tls/Readme.md)
5. [Network Policies Foundation](networking/03-network-policies/Readme.md)
6. [Traffic Management Labs](networking/04-traffic-splitting/Readme.md)
7. [Zero Trust Networking](networking/07-zerotrust-networking/Readme.md)

## Notes

- [ingress-demo](ingress-demo/) is currently an empty folder and can be used for future ingress-focused labs.
- [networking/setup-cluster-cilium/kind-config.yaml](networking/setup-cluster-cilium/kind-config.yaml) and [networking/setup-cluster-cilium/kind-config-cilium.yaml](networking/setup-cluster-cilium/kind-config-cilium.yaml) provide extra local cluster configuration variants.
