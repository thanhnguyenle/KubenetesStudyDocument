# Selectors

Selectors in Kubernetes are powerful mechanisms for identifying and filtering objects.

## 1. Label Selectors

Label selectors are the most commonly used type. They filter objects based on key-value pairs you assign as labels.

**How labels work:**
- Labels are metadata attached to objects (pods, services, deployments, etc.)
- They're defined in the manifest file under `metadata.labels`
- Format: `key: value` pairs

**Example manifest:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    environment: production
    tier: frontend
    owner: devops
```

**Using label selectors:**
- View labels: `kubectl get pods --show-labels`
- Filter by label: `kubectl get pods -l environment=production`
- Multiple conditions: `kubectl get pods -l environment=production,tier=frontend`
- Add labels to existing objects: `kubectl label pods apache-web owner=devops`

**Operators available:**
- Equality-based: `environment=production`, `tier!=backend`
- Set-based: `environment in (production,staging)`, `tier notin (cache)`

## 2. Field Selectors

Field selectors query objects based on their actual resource fields like metadata or status.

**Common field selector examples:**
- `metadata.name=my-pod` - select by name
- `metadata.namespace=default` - select by namespace
- `status.phase=Running` - select running pods
- `spec.nodeName=node-1` - select pods on specific node

**Usage:**
```bash
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default,status.phase=Pending
```

**Limitation:** Only certain fields are supported (varies by resource type), unlike labels which you can freely define.

## 3. Node Selectors

Node selectors are specifically for scheduling pods onto particular nodes based on node labels.

**How it works:**
1. Label your nodes: `kubectl label nodes worker-1 disk=ssd`
2. Add `nodeSelector` to your pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  nodeSelector:
    disk: ssd
    environment: production
  containers:
  - name: app
    image: nginx
```

This pod will only be scheduled on nodes that have both `disk=ssd` AND `environment=production` labels.