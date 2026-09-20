# Storage

## Storage Architecture

```
Application
    |
    v
PersistentVolumeClaim (PVC)
    |
    v
PersistentVolume (PV)
    |
    v
NFS Server (192.168.1.150)
```

## Storage Components

| Component | Purpose |
|-----------|---------|
| PersistentVolume (PV) | Cluster-level storage resource |
| PersistentVolumeClaim (PVC) | Request for storage by pod |
| StorageClass | Dynamic provisioning (not used yet) |
| NFS | Network File System server |

## Current Storage Setup

### NFS Server

| Property | Value |
|----------|-------|
| Server | 192.168.1.150 |
| Path | /exports |
| Access | ReadWriteOnce |

### PersistentVolumes

| Name | Capacity | Path | Status |
|------|----------|------|--------|
| grafana-pv | 5Gi | /exports/grafana | Bound |
| grafana-pv-nfs | 5Gi | /exports/grafana-nfs | Bound |
| postgres-pv | 5Gi | /exports/postgres-data | Bound |

### PersistentVolumeClaims

| Name | Namespace | Capacity | Status |
|------|-----------|----------|--------|
| grafana-pvc | monitoring | 5Gi | Bound |
| postgres-pvc | fullstack | 5Gi | Bound |

## Application Storage

### PostgreSQL

```yaml
# postgres-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 192.168.1.150
    path: /exports/postgres-data
```

```yaml
# postgres-statefulset.yaml
volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
```

### Grafana

```yaml
# grafana-deployment.yaml
volumes:
  - name: grafana-storage
    persistentVolumeClaim:
      claimName: grafana-pvc
```

## Storage Flow

```
1. Pod needs storage
    |
    v
2. Pod references PVC
    |
    v
3. PVC binds to PV
    |
    v
4. PV mounts NFS directory
    |
    v
5. Pod has persistent storage
```

## Data Persistence

| Component | Data Stored | Persistence |
|-----------|-------------|-------------|
| PostgreSQL | Database data | Persistent (NFS) |
| Grafana | Dashboards, configs | Persistent (NFS) |
| Prometheus | Metrics data | Persistent (hostPath) |

## Backup Strategy

### PostgreSQL Backup

```bash
# Backup database
kubectl exec -it postgres-0 -n fullstack -- pg_dump -U postgres fullstack > backup.sql

# Restore database
kubectl exec -it postgres-0 -n fullstack -- psql -U postgres fullstack < backup.sql
```

### NFS Backup

```bash
# On NFS server
tar -czf grafana-backup.tar.gz /exports/grafana
tar -czf postgres-backup.tar.gz /exports/postgres-data
```

## Troubleshooting Storage Issues

```bash
# Check PV status
kubectl get pv

# Check PVC status
kubectl get pvc -A

# Check if PVC is bound
kubectl describe pvc <pvc-name> -n <namespace>

# Check pod volume mounts
kubectl describe pod <pod-name> -n <namespace>

# Check NFS server connectivity
showmount -e 192.168.1.150

# Check mount on node
mount | grep nfs
```

## Common Issues

### PVC Pending

```
Status: Pending
```

**Cause:** No PV with matching capacity or access mode.

**Fix:** Create a PV with sufficient capacity.

### Pod Stuck in ContainerCreating

```
Warning: FailedMount: Unable to mount volumes
```

**Cause:** NFS server unreachable or path incorrect.

**Fix:** Verify NFS server is running and path is exported.

### Data Loss After Pod Restart

**Cause:** Using emptyDir instead of PVC.

**Fix:** Use PVC for persistent data.
