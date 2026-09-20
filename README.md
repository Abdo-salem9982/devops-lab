# DevOps Lab

Production-like DevOps/DevSecOps environment with Kubernetes, Argo CD, GitHub Actions, and a full-stack application.

## Architecture

```
Windows 11 Host (16GB RAM)
└── VMware
    ├── master (Control Plane) - 3.8GB RAM
    └── worker (Worker Node)
```

## Components

| Component | Purpose |
|-----------|---------|
| Kubernetes v1.30 | Container orchestration |
| Flannel | CNI network plugin |
| MetalLB | LoadBalancer for bare-metal |
| Traefik | Ingress controller |
| cert-manager | TLS certificate management |
| Argo CD | GitOps continuous delivery |
| Prometheus | Monitoring |
| Grafana | Dashboards |
| GitHub Actions | CI/CD pipeline |
| NFS | Persistent storage |

## Full-Stack Application

```
React + Vite + TailwindCSS (Frontend)
        |
        v
Node.js + Express (Backend)
        |
        v
PostgreSQL (Database)
```

## Project Structure

```
devops-lab/
├── app/                    # Application source code
│   ├── backend/            # Node.js + Express
│   └── frontend/           # React + Vite
│
├── argocd/                 # Argo CD (App of Apps)
│   ├── applications.yaml   # Root application
│   └── applications/       # Child applications
│
├── kubernetes/
│   ├── applications/       # Full-stack app manifests
│   └── infrastructure/     # Platform components
│
├── .github/workflows/      # GitHub Actions CI/CD
├── docs/                   # Documentation
└── screenshots/            # Project screenshots
```

## Quick Start

See [docs/setup.md](docs/setup.md) for detailed installation instructions.

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System architecture overview |
| [Setup](docs/setup.md) | Installation guide |
| [Networking](docs/networking.md) | Network configuration |
| [Storage](docs/storage.md) | Storage setup |
| [Troubleshooting](docs/troubleshooting.md) | Common issues |
| [Decisions](docs/decisions.md) | Architecture decisions |

## Accessing Services

| Service | URL |
|---------|-----|
| Frontend | https://frontend.local |
| Backend | https://backend.local |
| Argo CD | https://argo.local |
| Grafana | https://grafana.local |
| Prometheus | https://prometheus.local |

## CI/CD Pipeline

```
Push to GitHub
    |
    v
GitHub Actions
    ├── Build Docker images
    ├── Run tests
    ├── Security scan (Trivy)
    └── Update Kubernetes manifests
    |
    v
GitHub Container Registry (ghcr.io)
    |
    v
Argo CD syncs to Kubernetes
```

## Getting Started

1. Clone the repository
2. Follow [docs/setup.md](docs/setup.md)
3. Deploy the full-stack application
4. Access services via browser

## Technologies Used

- **Container Runtime:** containerd
- **Orchestration:** Kubernetes
- **Networking:** Flannel, MetalLB, Traefik
- **Storage:** NFS
- **GitOps:** Argo CD
- **CI/CD:** GitHub Actions
- **Monitoring:** Prometheus, Grafana
- **Security:** cert-manager, Trivy

## License

This project is for learning purposes.
