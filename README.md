# Cardano Node & DB-Sync Deployment on GKE

This repository contains Kubernetes manifests for deploying Cardano Node and DB-Sync on Google Kubernetes Engine (GKE), using the preprod testnet with Mithril bootstrapping for fast sync.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Deployment](#deployment)
- [Configuration](#configuration)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- GKE cluster is provisioned and running
- `kubectl` CLI installed and configured
- Storage classes available for persistent volumes
- Sufficient cluster resources

## Architecture

The deployment consists of the following components:

1. **Cardano Node**
   - Runs the core Cardano blockchain node
   - Uses Mithril for fast bootstrapping
   - Connects to preprod testnet

2. **DB-Sync**
   - Synchronizes blockchain data to PostgreSQL
   - Provides queryable database of blockchain data

3. **PostgreSQL**
   - Stores blockchain data
   - Optimized for DB-Sync workload

## Deployment

### 1. Clone the Repository
```bash
git clone <repository-url>
cd cardano-deployment
```

### 2. Deploy Components
Deploy in the following order:

```bash
# Create ConfigMap with genesis files
kubectl apply -f configmap.yaml

# Create Persistent Volume Claims
kubectl apply -f pvc.yaml

# Deploy PostgreSQL and DB-Sync
kubectl apply -f cardano-db-sync.yaml

# Deploy Cardano Node
kubectl apply -f cardano-node.yaml
```

## Configuration

### Storage Configuration (pvc.yaml)
```yaml
Persistent Volumes:
- mithril-snapshot: 100Gi (blockchain data)
- cardano-ipc: 100Gi (node-db-sync communication)
```

### Cardano Node Configuration (cardano-node.yaml)
```yaml
Key settings:
- Network: preprod
- Image: ghcr.io/intersectmbo/cardano-node:10.1.4
- Resources: 4Gi RAM, 1 CPU
- Mithril bootstrapping enabled
```

### DB-Sync Configuration (cardano-db-sync.yaml)
```yaml
Key settings:
- Image: ghcr.io/intersectmbo/cardano-db-sync:13.6.0.5
- PostgreSQL connection
- Shared IPC volume
```

### PostgreSQL Configuration
```yaml
Key settings:
- Image: postgres:17.2-alpine
- Resources: 2Gi RAM, 1 CPU
- Optimized for DB-Sync workload
```

### Network Configuration
```yaml
environment:
  NETWORK: preprod
  AGGREGATOR_ENDPOINT: https://aggregator.release-preprod.api.mithril.network/aggregator
```

## Monitoring

### Check Deployment Status
```bash
# View all pods
kubectl get pods

# Check Mithril bootstrap progress
kubectl logs -f <cardano-node-pod-name> -c mithril-client

# Monitor node synchronization
kubectl logs -f <cardano-node-pod-name> -c cardano-node

# Check DB-Sync progress
kubectl logs -f <cardano-db-sync-pod-name>
```

### Resource Monitoring
```bash
# Check resource usage
kubectl top pods

# Check PVC usage
kubectl get pvc
```

## Troubleshooting

### Common Issues

1. **Mithril Bootstrap Fails**
   - Check network connectivity
   - Verify storage permissions
   ```bash
   kubectl describe pod <cardano-node-pod-name>
   kubectl logs -f <cardano-node-pod-name> -c mithril-client
   ```

2. **Node Start-up Issues**
   - Verify IPC volume mounting
   - Check genesis configurations
   ```bash
   kubectl describe pod <cardano-node-pod-name>
   kubectl logs -f <cardano-node-pod-name> -c cardano-node
   ```

3. **DB-Sync Connection Issues**
   - Check PostgreSQL connection
   - Verify node socket file accessibility
   ```bash
   kubectl describe pod <cardano-db-sync-pod-name>
   kubectl logs -f <cardano-db-sync-pod-name>
   ```

### Resource Issues
```bash
# Check pod resource usage
kubectl top pods

# Check pod events
kubectl get events --sort-by='.lastTimestamp'
```

## Notes

- Adjust resource limits based on workload
- Implement proper monitoring for production
- Consider backup strategy for PostgreSQL
- Use Kubernetes secrets for sensitive data

## GKE-Specific Configuration

```yaml
tolerations:
  - key: app
    operator: Equal
    value: pipeline
```
