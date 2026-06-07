# Kubernetes Headless Service Demo (Kind / Local Lab)

In this hands-on lab, we will learn how **Headless Services** work in Kubernetes and how they enable **direct Pod-level DNS resolution**.

Unlike a regular Kubernetes Service, a **Headless Service does not provide a ClusterIP** and does **not load balance traffic**. Instead, Kubernetes returns the **individual Pod IPs** via DNS.

This is commonly used for:

- Stateful applications
- Databases
- Service discovery
- Direct Pod-to-Pod communication


---

# What We Will Build

In this demo, we will create:

- An **NGINX Deployment** with multiple Pods
- A **Headless Service**
- Verify **Pod-level DNS resolution**
- Inspect Kubernetes endpoints

Architecture:

```text
DNS Query
     ↓
Headless Service
(clusterIP: None)
     ↓
Direct Pod IPs Returned
     ↓
Pod 1    Pod 2    Pod 3
```

---

# Prerequisites

Before starting, make sure you have:

- A running Kubernetes cluster
- `kubectl` configured
- A local Kind cluster (recommended)

Verify cluster:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

---

# Step 1: Create NGINX Deployment

Create a file called `nginx-headless-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-headless

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx-headless

  template:
    metadata:
      labels:
        app: nginx-headless

    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f nginx-headless-deployment.yaml
```

Verify Pods:

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME                                READY   STATUS
nginx-headless-xxxxx                1/1     Running
nginx-headless-yyyyy                1/1     Running
nginx-headless-zzzzz                1/1     Running
```

You should see **3 running NGINX Pods**.

---

# Step 2: Create a Headless Service

Create a file called `nginx-headless-service.yaml`

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-headless

spec:
  clusterIP: None

  selector:
    app: nginx-headless

  ports:
    - port: 80
      targetPort: 80
```

Apply the Service:

```bash
kubectl apply -f nginx-headless-service.yaml
```

---

# Step 3: Verify the Service

Run:

```bash
kubectl get svc
```

Expected output:

```text
NAME                TYPE        CLUSTER-IP
nginx-headless      ClusterIP   None
```

Notice:

- `CLUSTER-IP = None`

This confirms the Service is a **Headless Service**.

### Why is it called "Headless"?

Normally, Kubernetes Services get a virtual IP (**ClusterIP**) that acts as a load balancer.

Example:

```text
App → Service IP → One of Many Pods
```

But a Headless Service skips this layer:

```text
App → DNS → Direct Pod IPs
```

There is **no virtual Service IP**.

---

# Step 4: Test DNS Resolution

Now we will verify Pod-level DNS discovery.

Launch a temporary test Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.35 \
  -it --rm -- sh
```

Inside the Pod, run:

```bash
nslookup nginx-headless
```

Expected result:

```text
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx-headless
Address 1: 10.244.1.5
Address 2: 10.244.1.6
Address 3: 10.244.1.7
```

Notice:

Instead of a **single Service IP**, Kubernetes returns **multiple Pod IPs**.

That is the magic of a **Headless Service** 🙂

Kubernetes directly exposes Pod endpoints through DNS.

---

# Step 5: Verify Endpoints

Exit the test Pod.

Now check endpoints:

```bash
kubectl get endpoints
```

Example output:

```text
NAME               ENDPOINTS
nginx-headless     10.244.1.5:80,10.244.1.6:80,10.244.1.7:80
```

You will notice all Pod IPs are listed individually.

You can also inspect EndpointSlices:

```bash
kubectl get endpointslices
```

Describe the EndpointSlice:

```bash
kubectl describe endpointslice
```

You should see all Pod IPs registered as healthy backends.

---

# Understanding the Flow

With a normal Service:

```text
Client
   ↓
ClusterIP Service
   ↓
Load Balancing
   ↓
Pods
```

With a Headless Service:

```text
Client
   ↓
DNS Lookup
   ↓
Multiple Pod IPs Returned
   ↓
Client Chooses Pod
```

Kubernetes DNS provides **direct Pod discovery**.

---

# Common Use Cases

Headless Services are commonly used with:

| Use Case | Why Headless Helps |
|----------|--------------------|
| Databases | Direct replica communication |
| StatefulSets | Stable Pod identity |
| Distributed systems | Pod-level service discovery |
| Messaging systems | Direct broker communication |

