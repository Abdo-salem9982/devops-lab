# Architecture Overview

## Project Architecture

```mermaid
flowchart TB
    subgraph DEV["Developer Workstation"]
        Code[Write Code]
        Git[Git Push]
    end

    subgraph GITHUB["GitHub Repository"]
        Repo[devops-lab Repository]
        Actions[GitHub Actions CI/CD]
        Registry[GitHub Container Registry<br/>ghcr.io]
    end

    subgraph PIPELINE["CI/CD Pipeline Stages"]
        Build[Build Docker Images]
        Test[Run Tests]
        Scan[Security Scan<br/>Trivy]
        Update[Update K8s Manifests]
    end

    subgraph VMware["VMware - Windows 11 Host"]
        subgraph MASTER["Master Node - 192.168.1.100"]
            API[kube-apiserver]
            ETCD[etcd]
            SCHED[kube-scheduler]
            CTRL[kube-controller-manager]
            ARGOCD[Argo CD]
        end

        subgraph WORKER["Worker Node - 192.168.1.101"]
            subgraph INFRA["Infrastructure Components"]
                TRAEFIK[Traefik<br/>Ingress Controller]
                METALLB[MetalLB<br/>LoadBalancer]
                CERTMGR[cert-manager<br/>TLS Certificates]
                PROM[Prometheus<br/>Monitoring]
                GRAFANA[Grafana<br/>Dashboards]
            end

            subgraph APP["Full-Stack Application"]
                FRONTEND[Frontend<br/>React + Vite]
                BACKEND[Backend<br/>Node.js + Express]
                POSTGRES[PostgreSQL<br/>Database]
            end

            subgraph STORAGE["Storage"]
                NFS[NFS Server<br/>192.168.1.150]
            end
        end

        subgraph NETWORK["Networking"]
            FLANNEL[Flannel CNI]
            DNS[CoreDNS]
        end
    end

    subgraph USERS["Users"]
        BROWSER[Browser]
    end

    %% CI/CD Flow
    Code --> Git
    Git --> Repo
    Repo --> Actions
    Actions --> Build
    Build --> Test
    Test --> Scan
    Scan --> Update
    Update --> Registry
    Update --> Repo

    %% GitOps Flow
    Repo --> ARGOCD
    ARGOCD --> API

    %% Kubernetes Deployment
    API --> WORKER
    FLANNEL --> WORKER
    DNS --> WORKER

    %% Application Flow
    METALLB --> TRAEFIK
    TRAEFIK --> FRONTEND
    TRAEFIK --> BACKEND
    BACKEND --> POSTGRES
    POSTGRES --> NFS

    %% Monitoring
    PROM --> GRAFANA
    PROM --> WORKER

    %% User Access
    BROWSER --> METALLB
    METALLB --> TRAEFIK

    %% Styling
    classDef devops fill:#2196F3,stroke:#1565C0,color:#fff
    classDef github fill:#24292e,stroke:#000,color:#fff
    classDef pipeline fill:#FF9800,stroke:#E65100,color:#fff
    classDef k8s fill:#326CE5,stroke:#1A4D8F,color:#fff
    classDef infra fill:#764ABC,stroke:#5C2D91,color:#fff
    classDef app fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef storage fill:#795548,stroke:#4E342E,color:#fff
    classDef network fill:#00BCD4,stroke:#00838F,color:#fff
    classDef user fill:#FF5722,stroke:#D84315,color:#fff

    class Code,Git devops
    class Repo,Actions,Registry github
    class Build,Test,Scan,Update pipeline
    class API,ETCD,SCHED,CTRL,ARGOCD,FLANNEL,DNS k8s
    class TRAEFIK,METALLB,CERTMGR,PROM,GRAFANA infra
    class FRONTEND,BACKEND,POSTGRES app
    class NFS storage
    class BROWSER user
```

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

```mermaid
flowchart TB
    A[Internet] --> B[MetalLB<br/>192.168.1.200-220]
    B --> C[Traefik<br/>Ingress Controller]
    C --> D[IngressRoute<br/>Host-based routing]
    D --> E[Kubernetes Service<br/>ClusterIP]
    E --> F[Pod<br/>CNI - Flannel]

    classDef internet fill:#FF5722,stroke:#D84315,color:#fff
    classDef infra fill:#764ABC,stroke:#5C2D91,color:#fff
    classDef k8s fill:#326CE5,stroke:#1A4D8F,color:#fff

    class A internet
    class B,C,D infra
    class E,F k8s
```

## Application Flow

```mermaid
flowchart LR
    A[Browser] -->|https://frontend.local| B[MetalLB<br/>192.168.1.200]
    B --> C[Traefik<br/>Ingress Controller]
    C -->|Host-based routing| D[Frontend<br/>React + Vite]
    D -->|/api/| E[Backend<br/>Node.js:5000]
    E -->|port 5432| F[PostgreSQL<br/>Database]
    F --> G[NFS Storage<br/>192.168.1.150]

    classDef user fill:#FF5722,stroke:#D84315,color:#fff
    classDef infra fill:#764ABC,stroke:#5C2D91,color:#fff
    classDef app fill:#4CAF50,stroke:#2E7D32,color:#fff
    classDef storage fill:#795548,stroke:#4E342E,color:#fff

    class A user
    class B,C infra
    class D,E,F app
    class G storage
```

## GitOps Workflow

```mermaid
flowchart TB
    A[Developer<br/>Push to GitHub] --> B[GitHub Actions<br/>Build - Test - Scan - Deploy]
    B --> C[GitHub Container<br/>Registry - ghcr.io]
    B --> D[Update K8s<br/>Manifests in Git]
    D --> E[Argo CD<br/>Detects Changes]
    E --> F[Argo CD<br/>Syncs to Kubernetes]
    F --> G[Application<br/>Updated]

    classDef dev fill:#2196F3,stroke:#1565C0,color:#fff
    classDef github fill:#24292e,stroke:#000,color:#fff
    classDef pipeline fill:#FF9800,stroke:#E65100,color:#fff
    classDef k8s fill:#326CE5,stroke:#1A4D8F,color:#fff
    classDef app fill:#4CAF50,stroke:#2E7D32,color:#fff

    class A dev
    class C,D github
    class B pipeline
    class E,F k8s
    class G app
```
