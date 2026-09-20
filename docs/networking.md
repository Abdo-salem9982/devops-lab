# Networking

## Network Architecture

```
External Traffic (192.168.1.200-220)
    |
    v
MetalLB (BGP/L2 Advertisement)
    |
    v
Traefik (Ingress Controller)
    |
    v
IngressRoute (Host-based routing)
    |
    v
Kubernetes Service (ClusterIP)
    |
    v
Pod (CNI - Flannel)
```

## IP Address Allocation

| Component | IP/Range | Purpose |
|-----------|----------|---------|
| Master Node | 192.168.1.100 | Control plane |
| Worker Node | 192.168.1.101 | Workloads |
| MetalLB Pool | 192.168.1.200-220 | External IPs |
| Pod CIDR | 10.244.0.0/16 | Pod IPs |
| Service CIDR | 10.96.0.0/12 | Service IPs |

## DNS Resolution

| Hostname | Resolves To | Routes To |
|----------|-------------|-----------|
| frontend.local | 192.168.1.200 | frontend-service:80 |
| backend.local | 192.168.1.200 | backend-service:5000 |
| argo.local | 192.168.1.200 | argocd-server:443 |
| grafana.local | 192.168.1.200 | grafana-service:3000 |
| prometheus.local | 192.168.1.200 | prometheus-service:9090 |

## Traffic Flow Examples

### Frontend Request

```
Browser (192.168.1.x)
    |
    v
DNS: frontend.local -> 192.168.1.200
    |
    v
MetalLB (192.168.1.200)
    |
    v
Traefik (receives HTTPS request)
    |
    v
IngressRoute: Host(`frontend.local`)
    |
    v
frontend-service (ClusterIP)
    |
    v
frontend-pod (10.244.x.x)
```

### API Request

```
Browser -> https://backend.local/api/...
    |
    v
MetalLB -> Traefik
    |
    v
IngressRoute: Host(`backend.local`) && PathPrefix(`/api`)
    |
    v
backend-service:5000
    |
    v
backend-pod
    |
    v
postgres-service:5432
    |
    v
postgres-pod
```

## Flannel CNI

Flannel provides pod networking by:
1. Assigning each node a subnet from the Pod CIDR
2. Creating virtual network interfaces
3. Encapsulating traffic between nodes (VXLAN)

```
Master (10.244.0.0/24)
    |-- pod-1 (10.244.0.x)
    |-- pod-2 (10.244.0.y)
    |
Worker (10.244.1.0/24)
    |-- pod-1 (10.244.1.x)
    |-- pod-2 (10.244.1.y)
```

## Network Policies

Currently no NetworkPolicies are applied. All pods can communicate with each other.

To add NetworkPolicies:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: fullstack
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

## Troubleshooting Network Issues

```bash
# Check pod IPs
kubectl get pods -o wide -A

# Test connectivity between pods
kubectl exec -it <pod> -- ping <target-ip>

# Check services
kubectl get svc -A

# Check endpoints
kubectl get endpoints -A

# Check ingressroutes
kubectl get ingressroute -A

# Check MetalLB
kubectl get pods -n metallb-system
kubectl get IPAddressPool -n metallb-system
```
