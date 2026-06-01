# PRACTICALS

## TOPIC 1 : Gateway API 
### Create NS
```bash
kubectl create ns infra-gateway
kubectl create ns site-a
kubectl create ns site-b
```

### Create Gateway 
`kubectl apply -f gateway.yaml --namespace=infra-gateway`
### To check 
`kubectl get gtw -n infra-gateway`

### Add workload to site a and b
# Deploy to Site A
`kubectl run web-a -n site-a --expose --port=5678 --image=hashicorp/http-echo -- -text="Welcome to Site A"`
# Deploy to Site B
`kubectl run web-b -n site-b --expose --port=5678 --image=hashicorp/http-echo -- -text="Welcome to Site B"`


### Because you are on a MacBook using Kind, there is no "Cloud Provider" to give you an External IP. To solve this for your hands-on without installing MetalLB, you have two choices:
### Option A (Recommended): Open a new terminal tab and run cloud-provider-kind. (Install it via brew install cloud-provider-kind). This will "bridge" Kind to your Mac.
### Option B: Use kubectl port-forward to the Gateway service Cilium created.

### We will go with option 1 cloud-provider-kind
# Install if you haven't yet
`brew install cloud-provider-kind`
# Run the provider (keep this tab open)
`sudo cloud-provider-kind`¯
# Now you will see IP
`kubectl get gtw -n infra-gateway`

# TOPIC 2 Header based routing
## Install http route
`kubectl apply -f http-route-site-a.yaml --namespace=site-a`
`kubectl apply -f http-route-site-b.yaml --namespace=site-b`

## TEST
# Get the IP
```bash
kubectl get gtw -n infra-gateway

export GATEWAY_IP=$(kubectl get gtw my-gateway -n infra-gateway -o jsonpath='{.status.addresses[0].value}')

# Test Default
curl -H "Host: claybrainer.lab" http://$GATEWAY_IP/weba
curl -H "Host: claybrainer.lab" http://$GATEWAY_IP/webb
```
