# Kubernetes LoadBalancer Service Demo on Kind

In this hands-on lab, we will learn how a **LoadBalancer Service** works in Kubernetes using **Kind (Kubernetes in Docker)**.

Since Kind runs Kubernetes locally inside Docker containers, it does **not provide a cloud LoadBalancer by default**. To simulate cloud-like LoadBalancer behavior, we will use **Cloud Provider KIND**, which enables `LoadBalancer` Services to work locally.

---

## Why Use Cloud Provider KIND?

In managed Kubernetes platforms, Kubernetes automatically provisions an external cloud load balancer whenever you create a `LoadBalancer` Service.

However, **Kind runs locally**, so there is no cloud infrastructure available to provision one.

**Cloud Provider KIND** bridges this gap by emulating LoadBalancer functionality locally, allowing us to test `LoadBalancer` Services in a development environment.

Traffic flow:

```text
Internet / Local Machine
          ↓
Cloud Provider KIND (LoadBalancer)
          ↓
NodePort
          ↓
ClusterIP Service
          ↓
Pods
```

---

# Prerequisites

Before starting, make sure you have:

- Docker installed
- `kubectl` installed
- Kind installed
- Cloud Provider KIND installed

---

# Step 1: Install Docker

You can refer here to install docker on to your machine. [Docker Setup](https://docs.docker.com/get-started/get-docker/)

---

# Step 2: Install kubectl

Install `kubectl`. [Guide](https://kubernetes.io/docs/tasks/tools/)

Verify:

```bash
kubectl version --client
```

---

# Step 3: Install Kind

Setting up Kind. [Guide](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

---

# Step 4: Install Cloud Provider KIND

Install Cloud Provider KIND. [Guide](https://github.com/kubernetes-sigs/cloud-provider-kind)



# Step 5: Create a Kind Cluster

Create a file called `kind-config.yaml`

```yaml
kind: Cluster # 
apiVersion: kind.x-k8s.io/v1alpha4 
nodes: 
- role: control-plane 
- role: worker 
- role: worker 
networking: 
  apiServerAddress: "0.0.0.0" 
```

Create the cluster:

```bash
kind create cluster --name lb-demo --config kind-config.yaml
```

Verify cluster:

```bash
kubectl cluster-info --context kind-lb-demo
```

Check nodes:

```bash
kubectl get nodes
```

---

# Step 6: Start Cloud Provider KIND

Run:

```bash
sudo cloud-provider-kind
```

Keep this terminal running.

This process watches for `LoadBalancer` Services and provisions a local external IP for them.

---

# Step 7: Create an NGINX Deployment

Create a file called `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx-app

  template:
    metadata:
      labels:
        app: nginx-app

    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Apply deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Verify Pods:

```bash
kubectl get pods -o wide
```

---

# Step 8: Create a LoadBalancer Service

Create a file called `nginx-loadbalancer.yaml`

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-lb

spec:
  type: LoadBalancer

  selector:
    app: nginx-app

  ports:
    - port: 80
      targetPort: 80
```

Apply Service:

```bash
kubectl apply -f nginx-loadbalancer.yaml
```

---

# Step 9: Verify the Service

Run:

```bash
kubectl get svc
```

Example output:

```text
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP
nginx-lb     LoadBalancer   10.96.10.50     172.xx.xx.xx
```

Notice:

- An **EXTERNAL-IP** is assigned
- A **NodePort** is automatically created internally

Verify details:

```bash
kubectl describe svc nginx-lb
```

Look for:

```text
Type: LoadBalancer
NodePort: 31xxx/TCP
Endpoints: 10.x.x.x:80
```

---

# Step 10: Access the Application

Get external IP:

```bash
kubectl get svc nginx-lb
```

Open in browser:

```bash
http://<EXTERNAL-IP>
```

You should see the **NGINX welcome page**.

You can also test using curl:

```bash
curl http://<EXTERNAL-IP>
```

---

# Step 11: Verify Traffic Routing

Check endpoints:

```bash
kubectl get endpoints
```

List EndpointSlices:

```bash
kubectl get endpointslices
```

Inspect Service:

```bash
kubectl describe svc nginx-lb
```

Observe Pods:

```bash
kubectl get pods -o wide
```

You will see the **pod IPs for healthy backends**. This is how Kubernetes knows where to route Service traffic.

---

# Understanding the Flow

When a request reaches your application:

```text
Browser Request
      ↓
External IP (Cloud Provider KIND)
      ↓
LoadBalancer Service
      ↓
NodePort
      ↓
ClusterIP Service
      ↓
NGINX Pods
```

Kubernetes automatically load balances requests across healthy Pods.

---

# Cleanup

Delete Service:

```bash
kubectl delete svc nginx-lb
```

Delete Deployment:

```bash
kubectl delete deployment nginx-app
```

Delete cluster:

```bash
kind delete cluster --name lb-demo
```

---

## Congratulations 🎉

You have successfully:

- Installed Docker, `kubectl`, and Kind
- Created a local Kubernetes cluster
- Configured **Cloud Provider KIND**
- Exposed an application using a **LoadBalancer Service**
- Understood how Kubernetes routes traffic internally
- Simulated cloud LoadBalancer behavior locally