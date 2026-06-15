# Deploy Three-Tier App (policy-lab)

This folder contains a simple three-tier demo application:

- `frontend` (Deployment + Service)
- `backend` (Deployment + Service)
- `database` (Deployment + Service)

All manifests are already configured with namespace `policy-lab`.

## Files

- `frontend.yaml`
- `backend.yaml`
- `db.yaml`

## Prerequisites

- A running Kubernetes cluster
- `kubectl` configured to your cluster
- Local cluster setup guide: [../../../setup-local-cluster/Readme.md](../../../setup-local-cluster/Readme.md)
- Cilium setup guide: [../../cilium/Readme.md](../../cilium/Readme.md)

## 1) Create Namespace

```bash
kubectl create namespace policy-lab
```

If it already exists, this is safe:

```bash
kubectl create namespace policy-lab --dry-run=client -o yaml | kubectl apply -f -
```

## 2) Deploy the App

From this folder (`networking/03-network-policies/app`):

```bash
kubectl apply -f db.yaml
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
```

## 3) Verify Resources

```bash
kubectl get deploy,svc,pods -n policy-lab
```

Expected services:

- `frontend`
- `backend`
- `database`

## 4) Basic Connectivity Test

Run test calls from the `frontend` pod:

```bash
# frontend -> backend
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://backend

# frontend -> database
kubectl exec -n policy-lab deploy/frontend -- curl --connect-timeout 3 -s -o /dev/null -w "%{http_code}\n" http://database
```

Because no policy is applied at this stage, both requests should normally return `200`.

## 5) Cleanup

```bash
kubectl delete -f frontend.yaml
kubectl delete -f backend.yaml
kubectl delete -f db.yaml

# Optional: remove namespace
kubectl delete namespace policy-lab
```
