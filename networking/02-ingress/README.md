# Kubernetes Ingress Controller Demo with Traefik on KIND

This demo provides a comprehensive end to end example of deploying and using an Ingress Controller with Traefik on a KIND (Kubernetes IN Docker) cluster. It demonstrates both path-based and host-based routing.

## Prerequisites

Make sure the following tools are available on your machine:

- **Docker Desktop on macOS/Windows, or Docker Engine on Linux**
- **KIND**
- **kubectl**
- **Helm** - Package manager for Kubernetes

If you do not have them installed yet, use the official setup pages below:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Docker Engine for Linux: https://docs.docker.com/engine/install/
- KIND: https://kind.sigs.k8s.io/docs/user/quick-start/
- kubectl: https://kubernetes.io/docs/tasks/tools/
- Helm: https://helm.sh/docs/intro/install/

You can verify the installation with:

```bash
docker --version
kind --version
kubectl version --client
helm version
```

## Demo Overview

This demo includes:

1. **KIND Cluster Setup** - A local Kubernetes cluster with port mappings for ingress
2. **Traefik Ingress Controller** - Modern HTTP reverse proxy and load balancer
3. **Two Sample Applications** - Simple HTTP echo servers (App1 and App2)
4. **Path-based Routing** - Route traffic based on URL paths (`/app1` and `/app2`)
5. **Host-based Routing** - Route traffic based on hostnames (`app1.local` and `app2.local`)

## Step-by-Step Guide

### Step 1: Create the KIND Cluster

Create a Kubernetes cluster using KIND with custom port mappings for ingress traffic:

```bash
# Navigate to the demo directory
cd networking/02-ingress

# Create the cluster
kind create cluster --config kind-cluster.yaml
```

**What this does:**
- Creates a 3-node Kubernetes cluster named `ingress-demo` (1 control-plane + 2 workers)
- Maps host ports 80 and 443 to the cluster for ingress traffic
- Takes approximately 2-3 minutes to complete

**Verify the cluster:**
```bash
# Check cluster status
kind get clusters

# Verify kubectl can connect to the cluster
kubectl cluster-info --context kind-ingress-demo

# Check nodes
kubectl get nodes
```

### Step 2: Install Helm and Deploy Traefik

First, add the Traefik Helm repository and install Traefik:

```bash
# Add Traefik Helm repository
helm repo add traefik https://traefik.github.io/charts

# Update Helm repositories
helm repo update

# Install Traefik using Helm
helm install traefik traefik/traefik --namespace traefik --create-namespace --set service.type=LoadBalancer --set service.ports.web.nodePort=30080 --set service.ports.websecure.nodePort=30443 --set dashboard.enabled=true --set dashboard.service.type=ClusterIP
```

**What this does:**
- Adds the official Traefik Helm repository
- Installs Traefik in the `traefik` namespace (creates it if it doesn't exist)
- Configures Traefik as a LoadBalancer service
- Enables the Traefik dashboard
- Sets up necessary RBAC permissions automatically via Helm

**Verify the deployment:**
```bash
# Check Traefik pod status
kubectl get pods -n traefik

# Check Traefik services
kubectl get svc -n traefik

# Check Helm release
helm list -n traefik

# Check IngressClass
kubectl get ingressclass
```

Expected output should show:
- Traefik pod in `Running` state
- LoadBalancer service with external IPs
- Helm release named `traefik`
- IngressClass named `traefik`

### Step 3: Deploy Sample Applications

Deploy two simple HTTP echo server applications:

```bash
# Deploy App1
kubectl apply -f app1-deployment.yaml

# Deploy App2
kubectl apply -f app2-deployment.yaml
```

**What this does:**
- Creates `app1` and `app2` namespaces
- Deploys 2 replicas of each application
- Creates ClusterIP services for each application
- Each app responds with a unique message identifying which app is being accessed

**Verify the deployments:**
```bash
# Check App1
kubectl get pods -n app1
kubectl get svc -n app1

# Check App2
kubectl get pods -n app2
kubectl get svc -n app2

# Test services directly from within the cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://app1-service.app1
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://app2-service.app2
```

### Step 4: Path-Based Routing

Deploy an Ingress that routes traffic based on URL paths:

```bash
# Apply path-based routing ingress
kubectl apply -f ingress-path-based.yaml
```

**What this does:**
- Creates an Ingress resource in the default namespace
- Routes `/app1/*` traffic to App1 service
- Routes `/app2/*` traffic to App2 service

**Verify the Ingress:**
```bash
# Check Ingress status
kubectl get ingress

# Get detailed Ingress information
kubectl describe ingress path-based-ingress
```

**Test path-based routing:**
```bash
# Access App1 via path /app1
curl http://localhost/app1

# Access App2 via path /app2
curl http://localhost/app2
```

Expected output:
- `/app1` returns: "Hello from App1! You are accessing the first application."
- `/app2` returns: "Hello from App2! You are accessing the second application."

### Step 5: Host-Based Routing

Deploy an Ingress that routes traffic based on hostnames:

```bash
# Apply host-based routing ingress
kubectl apply -f ingress-host-based.yaml
```

**What this does:**
- Creates an Ingress resource that routes based on host headers
- Routes `app1.local` to App1 service
- Routes `app2.local` to App2 service

**Configure local DNS:**
Add the following entries to your `/etc/hosts` file:

```bash
# Edit hosts file (requires sudo)
sudo nano /etc/hosts

# Add these lines:
127.0.0.1 app1.local
127.0.0.1 app2.local
```

**Verify the Ingress:**
```bash
# Check Ingress status
kubectl get ingress

# Get detailed Ingress information
kubectl describe ingress host-based-ingress
```

**Test host-based routing:**
```bash
# Access App1 via hostname
curl http://app1.local

# Access App2 via hostname
curl http://app2.local
```

Expected output:
- `app1.local` returns: "Hello from App1! You are accessing the first application."
- `app2.local` returns: "Hello from App2! You are accessing the second application."

### Step 6: Access Traefik Dashboard

Access the Traefik dashboard to monitor your ingress controller:

```bash
# Port-forward to access the dashboard
kubectl port-forward -n traefik svc/traefik-dashboard 8080:8080
```

Then open your browser and navigate to:
- http://localhost:8080/dashboard/

The dashboard shows:
- Active routers and services
- HTTP and HTTPS metrics
- Real-time traffic monitoring

## Understanding the Components

### KIND Configuration (`kind-cluster.yaml`)
- Defines a 3-node cluster (1 control-plane + 2 workers)
- Maps host ports 80 and 443 to the cluster for ingress
- Enables external access to services through the ingress controller

### Traefik Deployment (Helm Chart)
- **Helm Chart**: Uses the official Traefik Helm chart for easy deployment
- **Namespace**: Creates and isolates Traefik resources in the `traefik` namespace
- **ServiceAccount & RBAC**: Automatically configured by Helm
- **Deployment**: Runs Traefik with ingress provider enabled
- **LoadBalancer Service**: Exposes Traefik on ports 80, 443, and dashboard on 8080
- **IngressClass**: Defines the ingress controller for the cluster

### Sample Applications (`app1-deployment.yaml`, `app2-deployment.yaml`)
- **Deployment**: Runs 2 replicas of each application for high availability
- **Service**: Creates ClusterIP services for internal cluster access
- **Image**: Uses `hashicorp/http-echo` for simple HTTP responses

### Path-Based Ingress (`ingress-path-based.yaml`)
- Routes traffic based on URL path prefixes
- Uses `pathType: Prefix` for flexible path matching
- Example: `/app1` routes all requests starting with `/app1` to App1

### Host-Based Ingress (`ingress-host-based.yaml`)
- Routes traffic based on the Host header
- Requires DNS configuration (local /etc/hosts for testing)
- Example: `app1.local` routes all requests for that host to App1

## Troubleshooting

### Cluster won't start
```bash
# Delete and recreate the cluster
kind delete cluster --name ingress-demo
kind create cluster --config kind-cluster.yaml
```

### Traefik pod not starting
```bash
# Check pod logs
kubectl logs -n traefik deployment/traefik

# Check pod events
kubectl describe pod -n traefik -l app.kubernetes.io/name=traefik

# Check Helm release status
helm status traefik -n traefik
```

### Ingress not working
```bash
# Check Ingress status
kubectl get ingress
kubectl describe ingress <ingress-name>

# Check Traefik logs for routing errors
kubectl logs -n traefik deployment/traefik --tail=50
```

### Services not accessible
```bash
# Check service endpoints
kubectl get endpoints -n <namespace>

# Test service connectivity from within cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://<service-name>.<namespace>
```

### Port conflicts
If ports 80 or 443 are already in use on your host:
```bash
# Check what's using the ports
sudo lsof -i :80
sudo lsof -i :443

# Modify kind-cluster.yaml to use different host ports
```

## Cleanup

To remove all resources and delete the cluster:

```bash

# Delete KIND cluster
kind delete cluster --name ingress-demo

# Remove /etc/hosts entries (if added)
sudo nano /etc/hosts
# Remove the lines: 127.0.0.1 app1.local and 127.0.0.1 app2.local
```

## Key Concepts Learned

1. **Ingress Controller**: A specialized load balancer that manages ingress rules
2. **Ingress Resource**: Defines routing rules for HTTP/HTTPS traffic
3. **Path-based Routing**: Routes traffic based on URL paths
4. **Host-based Routing**: Routes traffic based on domain/hostnames
5. **IngressClass**: Defines which ingress controller handles specific ingress resources
6. **Service Discovery**: Ingress automatically discovers services via the Kubernetes API

## Next Steps

- **TLS/SSL**: Add HTTPS support with TLS certificates
- **Authentication**: Implement basic auth or OAuth with Traefik middleware
- **Rate Limiting**: Add rate limiting to protect your services
- **Canary Deployments**: Use weighted routing for canary releases
- **Monitoring**: Integrate Prometheus and Grafana for metrics

## Additional Resources

- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [KIND Documentation](https://kind.sigs.k8s.io/)
- [Traefik Kubernetes Ingress Provider](https://doc.traefik.io/traefik/providers/kubernetes-ingress/)

## License

This demo is provided for educational purposes.
