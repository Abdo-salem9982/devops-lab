# Troubleshooting

## Common Issues and Solutions

### 1. Pods Stuck in Pending

**Symptom:**
```
NAME                    READY   STATUS    RESTARTS   AGE
my-pod                  0/1     Pending   0          5m
```

**Diagnosis:**
```bash
kubectl describe pod <pod-name>
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Insufficient resources | Add more RAM/CPU to node |
| PVC not bound | Check PV/PVC status |
| Node not ready | Check node status |
| Taints/tolerations | Add tolerations to pod |

**Fix:**
```bash
# Check node resources
kubectl describe node worker | grep -A 5 "Allocated resources"

# Check PVC status
kubectl get pvc -A

# Check node status
kubectl get nodes
```

---

### 2. Pods in CrashLoopBackOff

**Symptom:**
```
NAME                    READY   STATUS             RESTARTS   AGE
my-pod                  0/1     CrashLoopBackOff   5          10m
```

**Diagnosis:**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Application error | Check application logs |
| Missing config | Verify ConfigMap/Secret |
| Wrong command | Check deployment spec |
| Health check failing | Increase probe delays |

**Fix:**
```bash
# Check logs
kubectl logs <pod-name>

# Check previous container logs
kubectl logs <pod-name> --previous

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

### 3. Service Cannot Connect to Pod

**Symptom:**
- Service exists but endpoints are empty
- Application timeout errors

**Diagnosis:**
```bash
kubectl get endpoints <service-name>
kubectl describe service <service-name>
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Selector mismatch | Fix service selector |
| Pod not ready | Fix readiness probe |
| Wrong port | Check service port |
| Pod in different namespace | Use cross-namespace service |

**Fix:**
```bash
# Check service selector
kubectl get svc <service-name> -o yaml

# Check pod labels
kubectl get pods --show-labels

# Verify pod is ready
kubectl get pods -o wide
```

---

### 4. Ingress Not Working

**Symptom:**
- 404 or 502 errors when accessing URL
- Connection refused

**Diagnosis:**
```bash
kubectl get ingressroute -A
kubectl describe ingressroute <name> -n <namespace>
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Wrong host | Check IngressRoute host |
| Wrong service | Verify service name/port |
| TLS issue | Check certificate |
| Traefik not running | Check Traefik pods |

**Fix:**
```bash
# Check IngressRoutes
kubectl get ingressroute -A

# Check Traefik pods
kubectl get pods -n traefik

# Check certificate
kubectl get certificate -A
```

---

### 5. Argo CD Application OutOfSync

**Symptom:**
```
NAME              SYNC STATUS   HEALTH STATUS
my-app            OutOfSync     Healthy
```

**Diagnosis:**
```bash
kubectl describe application <app-name> -n argocd
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Manual changes | Let Argo CD self-heal |
| Git not updated | Push changes to Git |
| Permission issue | Check RBAC |
| Repo server down | Restart repo-server |

**Fix:**
```bash
# Force sync
kubectl patch application <app-name> -n argocd --type merge -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'

# Refresh
kubectl patch application <app-name> -n argocd --type merge -p '{"spec":{"source":{"repoURL":"<url>"}}}'
```

---

### 6. Memory Pressure

**Symptom:**
- Pods being OOMKilled
- Slow startup times
- Node NotReady

**Diagnosis:**
```bash
free -h
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Too many pods | Reduce pod count |
| High memory usage | Add more RAM |
| No resource limits | Set resource limits |
| Memory leak | Fix application |

**Fix:**
```bash
# Check memory usage
free -h

# Check pod resource requests
kubectl get pods -o custom-columns='NAME:.metadata.name,MEMORY_REQ:.spec.containers[*].resources.requests.memory'

# Add resource limits
kubectl patch deployment <name> -n <namespace> --type json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/resources/limits/memory", "value": "256Mi"}]'
```

---

### 7. TLS Certificate Issues

**Symptom:**
- Browser shows "Not Secure"
- Certificate warnings

**Diagnosis:**
```bash
kubectl get certificate -A
kubectl describe certificate <name> -n <namespace>
```

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Certificate not issued | Check cert-manager |
| Wrong hostname | Match IngressRoute host |
| Expired certificate | Renew certificate |
| Issuer not ready | Check issuer status |

**Fix:**
```bash
# Check certificates
kubectl get certificate -A

# Check issuers
kubectl get issuer -A

# Check certificate details
kubectl describe certificate <name> -n <namespace>
```

---

## Debug Commands Reference

```bash
# Pod debugging
kubectl get pods -A
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Service debugging
kubectl get svc -A
kubectl get endpoints -A
kubectl describe svc <service-name> -n <namespace>

# Network debugging
kubectl get ingressroute -A
kubectl get certificate -A
kubectl get pods -n traefik

# Storage debugging
kubectl get pv
kubectl get pvc -A
kubectl describe pvc <pvc-name> -n <namespace>

# Node debugging
kubectl get nodes
kubectl describe node <node-name>
free -h
df -h

# Argo CD debugging
kubectl get applications -n argocd
kubectl describe application <app-name> -n argocd
kubectl get pods -n argocd
```

## Escalation Steps

1. Check pod status and events
2. Check application logs
3. Check service/endpoints
4. Check network connectivity
5. Check node resources
6. Check cluster events
7. Check component logs
