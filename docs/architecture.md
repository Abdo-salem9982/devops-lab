# Architecture Overview

## Lab Environment

```
Windows 11 Host (16GB RAM)
└── VMware
    ├── master (Control Plane) - 3.8GB RAM
    │   ├── kube-apiserver
    │   ├── etcd
    │   ├── kube-scheduler
    │   ├── kube-controller-manager
    │   └── Argo CD
    │
    └── worker (Worker Node)
        ├── Full-stack Application
        │   ├── Frontend (React + Vite)
        │   ├── Backend (Node.js + Express)
        │   └── PostgreSQL
        │
        ├── Infrastructure
        │   ├── Traefik (Ingress Controller)
        │   ├── MetalLB (LoadBalancer)
        │   ├── cert-manager (TLS)
        │   ├── Prometheus (Monitoring)
        │   └── Grafana (Dashboards)
        │
        └── Core Components
            ├── Flannel (CNI)
            ├── containerd (Runtime)
            └── CoreDNS (DNS)
```

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Container Runtime | containerd | Runs containers |
| CNI | Flannel | Pod networking |
| Ingress | Traefik | HTTP routing |
| LoadBalancer | MetalLB | External IPs |
| TLS | cert-manager | Certificate management |
| GitOps | Argo CD | Continuous delivery |
| Monitoring | Prometheus | Metrics collection |
| Visualization | Grafana | Dashboards |
| CI/CD | GitHub Actions | Build and deploy |

## Network Flow

```
Internet
    |
    v
MetalLB (192.168.1.200-220)
    |
    v
Traefik (Ingress Controller)
    |
    v
IngressRoute (Host-based routing)
    |
    v
Kubernetes Service
    |
    v
Pod
```

## Application Flow

```
Browser -> https://frontend.local
    |
    v
Frontend (React) -> /api/ -> Backend (Node.js:5000)
    |
    v
Backend -> PostgreSQL (port 5432)
    |
    v
Database Connection Successful
```

## GitOps Workflow

```
Developer pushes to GitHub
    |
    v
GitHub Actions (Build -> Test -> Scan -> Deploy)
    |
    v
GitHub Container Registry (ghcr.io)
    |
    v
Argo CD detects changes
    |
    v
Argo CD syncs to Kubernetes
    |
    v
Application updated
```
