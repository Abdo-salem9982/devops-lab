# Architecture Decisions

## Overview

This document records the key architectural decisions made during the DevOps lab rebuild, including the reasoning behind each choice.

---

## 1. Git Hosting: GitLab vs GitHub

**Decision:** Use GitHub for repository, GitHub Actions for CI/CD.

**Options Considered:**
- GitLab.com (original plan)
- GitHub with GitHub Actions
- Self-hosted GitLab CE

**Reasoning:**
- GitHub is the most popular platform for open-source and portfolios
- GitHub Actions is widely used in industry
- Single platform (no need to manage two systems)
- Better for CV visibility

**Trade-offs:**
- GitLab has built-in container registry (GitHub uses ghcr.io)
- GitLab CI/CD is more mature for some use cases

---

## 2. Kubernetes Installation: Manual vs Automated

**Decision:** Manual installation using kubeadm.

**Options Considered:**
- kubeadm (manual)
- k3s (lightweight)
- minikube (single-node)
- Rancher (GUI-based)

**Reasoning:**
- kubeadm is the standard Kubernetes installer
- Teaches how Kubernetes components work
- Required for CKA/CKAD certification knowledge
- Full control over cluster configuration

**Trade-offs:**
- More time-consuming than k3s/minikube
- Requires more resources
- More complex to troubleshoot

---

## 3. CNI: Flannel vs Calico vs Cilium

**Decision:** Flannel.

**Options Considered:**
- Flannel
- Calico
- Cilium

**Reasoning:**
- Simplest to install and configure
- Sufficient for lab environment
- Good documentation
- Low resource usage

**Trade-offs:**
- No network policy support (Calico/Cilium have this)
- Less features than Calico/Cilium
- VXLAN encapsulation adds overhead

---

## 4. Ingress: Traefik vs Nginx vs HAProxy

**Decision:** Traefik.

**Options Considered:**
- Traefik
- Nginx Ingress Controller
- HAProxy

**Reasoning:**
- Native Kubernetes integration
- Automatic service discovery
- IngressRoute CRD (better than raw Ingress)
- Built-in dashboard
- Let's Encrypt integration

**Trade-offs:**
- Less mature than Nginx
- Smaller community
- Configuration can be complex

---

## 5. LoadBalancer: MetalLB vs Cloud Provider

**Decision:** MetalLB.

**Options Considered:**
- MetalLB
- Cloud provider load balancer
- NodePort

**Reasoning:**
- Only option for bare-metal clusters
- Simulates cloud load balancer behavior
- Provides real external IPs
- L2 mode is simple to configure

**Trade-offs:**
- Requires IP address pool management
- No cloud integration
- L2 mode has single-point-of-failure

---

## 6. TLS: cert-manager vs Self-Signed

**Decision:** cert-manager with self-signed CA.

**Options Considered:**
- cert-manager with self-signed CA
- cert-manager with Let's Encrypt
- Manual certificate management
- No TLS

**Reasoning:**
- cert-manager is the Kubernetes standard
- Self-signed CA works without internet
- Automatic certificate renewal
- Teaches certificate management

**Trade-offs:**
- Self-signed certificates show warnings in browsers
- Let's Encrypt requires public DNS
- More complex than manual certificates

---

## 7. GitOps: Argo CD vs Flux

**Decision:** Argo CD.

**Options Considered:**
- Argo CD
- Flux CD
- Manual deployment

**Reasoning:**
- Better UI/UX
- App of Apps pattern
- Strong community
- More features for multi-cluster
- Better for learning

**Trade-offs:**
- More resource-intensive than Flux
- Steeper learning curve
- Requires more cluster resources

---

## 8. Monitoring: Prometheus + Grafana vs alternatives

**Decision:** Prometheus + Grafana.

**Options Considered:**
- Prometheus + Grafana
- Datadog (SaaS)
- New Relic (SaaS)
- ELK Stack

**Reasoning:**
- Industry standard for Kubernetes monitoring
- Open source and free
- Extensive ecosystem
- Good documentation
- Teaches monitoring concepts

**Trade-offs:**
- Requires more resources than SaaS
- Self-managed (no vendor support)
- Complex to scale

---

## 9. Storage: NFS vs Local vs Cloud

**Decision:** NFS for persistent storage.

**Options Considered:**
- NFS
- Local path provisioner
- Cloud storage (EBS, GCE PD)
- Longhorn

**Reasoning:**
- NFS is simple to set up
- Supports ReadWriteMany
- Data persists across node reboots
- Can be expanded easily
- Works on bare metal

**Trade-offs:**
- Single point of failure (NFS server)
- Performance limitations
- No dynamic provisioning
- Manual PV creation required

---

## 10. Container Runtime: containerd vs Docker

**Decision:** containerd.

**Options Considered:**
- containerd
- Docker (dockershim)
- CRI-O

**Reasoning:**
- Docker deprecated in Kubernetes 1.24+
- containerd is the industry standard
- Lower resource usage
- Direct CRI implementation

**Trade-offs:**
- Less tooling than Docker
- No Docker CLI
- Different debugging approach

---

## 11. Helm: Used vs Not Used

**Decision:** No Helm (plain YAML manifests).

**Options Considered:**
- Helm charts
- Plain YAML manifests
- Kustomize

**Reasoning:**
- Teaches raw Kubernetes manifests
- Better understanding of resources
- No Helm dependency
- Simpler for learning

**Trade-offs:**
- More verbose than Helm
- No templating
- Harder to manage complex applications
- More files to maintain

---

## 12. Argo CD Application Management: App of Apps

**Decision:** App of Apps pattern.

**Options Considered:**
- App of Apps pattern
- ApplicationSets
- Manual application management

**Reasoning:**
- Full GitOps workflow
- Adding new apps is just a git push
- Automatic sync and self-heal
- Industry standard pattern

**Trade-offs:**
- More complex than manual management
- Requires understanding of Argo CD
- Root application is critical

---

## Summary Table

| Decision | Choice | Reason |
|----------|--------|--------|
| Git Hosting | GitHub | Portfolio visibility |
| K8s Installation | kubeadm | Learning value |
| CNI | Flannel | Simplicity |
| Ingress | Traefik | K8s native |
| LoadBalancer | MetalLB | Bare metal support |
| TLS | cert-manager | Industry standard |
| GitOps | Argo CD | Better UI/features |
| Monitoring | Prometheus + Grafana | Industry standard |
| Storage | NFS | Simplicity |
| Container Runtime | containerd | K8s standard |
| Templating | Plain YAML | Learning value |
| App Management | App of Apps | Full GitOps |
