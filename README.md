# kubernetes_learning

This repository helps you learn Kubernetes by doing.

The repository is structured so that each component has its own section, where you can learn the concept in detail and follow practical exercises. Each section includes its own README with the necessary explanations, steps, and commands, so you can follow along and practice as you learn.

## Getting Started

To set up your local lab for learning Kubernetes, start with the [setup-local-cluster](setup-local-cluster/Readme.md) section.

That section explains how to create a local Kind-based Kubernetes cluster and prepares the environment used by the examples in this repository.

After your cluster is ready, configure Cilium using [networking/cilium/Readme.md](networking/cilium/Readme.md).

## Topics

- Networking
    - Fundamentals
        - [Pod-to-Pod Communication](networking/01-fundamentals/01-pod-to-pod-communication.md)
        - [Pod and Service Communication](networking/01-fundamentals/02-pod-and-service-communication.md)
        - [LoadBalancer Service Practice](networking/01-fundamentals/03-LoadBalancerService-practice.md)
        - [Headless Service Practice](networking/01-fundamentals/04-headless-service.md)
    - [Ingress Practice (Traefik on Kind)](networking/02-ingress/README.md)
    - [Cilium Setup](networking/cilium/Readme.md)
    - [Gateway API Practice (Kind + Cilium)](networking/03-gateway-api/Readme.md)
    

## Suggested Learning Order

1. [Setup Local Kind Cluster](setup-local-cluster/Readme.md)
2. [Cilium Setup](networking/cilium/Readme.md)
3. [Networking Fundamentals](networking/01-fundamentals/01-pod-to-pod-communication.md)
4. [Ingress Practice](networking/02-ingress/README.md)
5. [Gateway API Practice](networking/03-gateway-api/Readme.md)
