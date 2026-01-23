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

# Labels

These are **standardized labels** that help organize, identify, and manage applications in Kubernetes. They follow a common naming convention with the `app.kubernetes.io/` prefix.

## The 7 Recommended Labels

### 1. `app.kubernetes.io/name`
**The name of the application**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: mysql
```

- The **application name** itself (e.g., mysql, wordpress, nginx)
- Should be the same across all instances of this app
- Use lowercase, no version info
---

### 2. `app.kubernetes.io/instance`
**A unique name for this specific instance**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-production
```

- Identifies **which deployment** of the application
- Useful when you have multiple instances (dev, staging, prod)
- Should be unique within your cluster/namespace
---

### 3. `app.kubernetes.io/version`
**Current version of the application**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/version: "5.8.1"
```

- Tracks the **application version** (not the chart or image tag necessarily)
- Use semantic versioning, git commit hash, or build number
- Helps with rollbacks and auditing
---

### 4. `app.kubernetes.io/component`
**The component within the architecture**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/component: database
```

Common values:
- `frontend`
- `backend`
- `database`
- `cache`
- `queue`
- `api`
---

### 5. `app.kubernetes.io/part-of`
**The higher-level application this belongs to**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/part-of: ecommerce-platform
```

- Groups related applications together
- Represents the **overall system** or **project**
---

### 6. `app.kubernetes.io/managed-by`
**Tool managing the application**

```yaml
metadata:
  labels:
    app.kubernetes.io/managed-by: helm
```

Common values:
- `helm`
- `kustomize`
- `argocd`
- `flux`
- `kubectl`
- `terraform`
---

### 7. `app.kubernetes.io/created-by`
**Who/what created this resource**

```yaml
metadata:
  labels:
    app.kubernetes.io/created-by: controller-manager
```

Values could be:
- `deployment-controller`
- `replicaset-controller`
- `user-john`
- `ci-pipeline`

---

## Querying with These Labels

```bash
# All resources for the blog platform
kubectl get all -l app.kubernetes.io/part-of=blog-platform

# Only production instances
kubectl get all -l app.kubernetes.io/instance=wordpress-prod

# All databases across all applications
kubectl get all -l app.kubernetes.io/component=database

# All resources managed by Helm
kubectl get all -l app.kubernetes.io/managed-by=helm

# Specific version of WordPress
kubectl get pods -l app.kubernetes.io/name=wordpress,app.kubernetes.io/version=6.4

# Frontend components of the blog platform
kubectl get all -l app.kubernetes.io/part-of=blog-platform,app.kubernetes.io/component=frontend
```

## Label Prefix Rules

### `app.kubernetes.io/` prefix
- **Shared/standard** labels
- Reserved for Kubernetes community
- Well-known meaning across tools and teams

### No prefix (e.g., `env`, `team`)
- **Private** to your organization
- Custom labels for your specific needs

```yaml
metadata:
  labels:
    # Standard labels
    app.kubernetes.io/name: mysql
    app.kubernetes.io/component: database
    
    # Your custom labels (no prefix)
    env: production
    team: platform
    cost-center: engineering
```

## Best Practices

1. **Always include at minimum:**
   - `app.kubernetes.io/name`
   - `app.kubernetes.io/instance`

2. **Use on ALL related resources:**
   - Deployments
   - Services
   - ConfigMaps
   - Secrets
   - PersistentVolumeClaims

3. **Keep values consistent:**
   - Use lowercase
   - Use hyphens, not underscores
   - No spaces

4. **Combine with selectors:**
```yaml
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: wordpress
      app.kubernetes.io/instance: wordpress-prod
```

These labels make your Kubernetes resources **self-documenting** and **easier to manage at scale**!

# Label Selectors

Label selectors allow Kubernetes objects to **find and target specific pods** based on their labels. Different Kubernetes resources use different selector syntaxes.

---
## Two Main Selector Syntaxes

### 1. **Equality-Based Selectors** (Simple)
Used by: **Services**, **ReplicationControllers**

```yaml
selector:
  app: frontend
  env: production
```

- Uses simple **key: value** matching
- All labels must match (AND logic)
- Older, simpler syntax

### 2. **Set-Based Selectors** (Advanced)
Used by: **ReplicaSets**, **Deployments**, **DaemonSets**, **Jobs**

```yaml
selector:
  matchLabels:
    app: frontend
  matchExpressions:
    - key: env
      operator: In
      values:
        - production
        - staging
```

- More powerful and flexible
- Supports operators: `In`, `NotIn`, `Exists`, `DoesNotExist`
- Can combine multiple conditions

---

## Set-Based Operators

### `In` - Value must be in the list
```yaml
matchExpressions:
  - key: env
    operator: In
    values:
      - production
      - staging
```
Matches: `env=production` OR `env=staging`

### `NotIn` - Value must NOT be in the list
```yaml
matchExpressions:
  - key: env
    operator: NotIn
    values:
      - development
      - testing
```
Matches: Any pod where `env` is NOT `development` or `testing`

### `Exists` - Label key must exist (any value)
```yaml
matchExpressions:
  - key: security
    operator: Exists
```
Matches: Any pod with a `security` label, regardless of value

### `DoesNotExist` - Label key must NOT exist
```yaml
matchExpressions:
  - key: deprecated
    operator: DoesNotExist
```
Matches: Pods that don't have a `deprecated` label

---

## Important Rules

### 1. **Pod template labels MUST match selector**
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  selector:
    matchLabels:
      app: frontend  # ← Selector
  template:
    metadata:
      labels:
        app: frontend  # ← MUST match selector
```

If they don't match, you'll get an error:
```
error: selector does not match template labels
```

### 2. **Selectors are immutable** (mostly)
Once created, you **cannot** change selectors for:
- Deployments
- ReplicaSets
- Services (can change, but not recommended)

You must delete and recreate the resource.

### 3. **Empty selector matches nothing**
```yaml
selector: {}  # Matches NO pods
```

---

## Summary Table

| Resource Type | Selector Syntax | Example |
|--------------|-----------------|---------|
| **Service** | Equality-based | `selector: {app: frontend}` |
| **ReplicationController** | Equality-based | `selector: {app: frontend}` |
| **ReplicaSet** | Set-based | `matchLabels: {app: frontend}` |
| **Deployment** | Set-based | `matchLabels: {app: frontend}` |
| **DaemonSet** | Set-based | `matchLabels: {app: frontend}` |
| **Job** | Set-based | `matchLabels: {app: frontend}` |
| **kubectl CLI** | Both | `-l app=frontend` or `-l 'env in (prod,staging)'` |

---

# Annotations

## What Are Annotations?

**Annotations = Notes you attach to Kubernetes objects**

They store extra information that Kubernetes doesn't use for selection, but tools and humans can read.

## Labels vs Annotations - Quick Comparison

```yaml
metadata:
  labels:
    app: frontend        # Kubernetes USES this to find/select objects
  annotations:
    note: "Production"   # Kubernetes IGNORES this (just stores it)
```

**Labels** = For Kubernetes to find things  
**Annotations** = For storing extra info

## Common Uses (4 Main Ones)

### 1. **Tool Configuration**
Tools read annotations to know how to behave:

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
  # Tells Nginx Ingress Controller to rewrite URLs
```

### 2. **Documentation**
Add notes for humans:

```yaml
annotations:
  description: "Main payment processing service"
  owner: "team-payments@company.com"
```

### 3. **Build Info**
Track deployment details:

```yaml
annotations:
  git-commit: "abc123"
  deployed-by: "jenkins"
  build-date: "2025-01-23"
```

### 4. **Monitoring Setup**
Configure monitoring tools:

```yaml
annotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "9090"
```
## Labels vs Annotations

| Aspect | Labels | Annotations |
|--------|--------|-------------|
| **Purpose** | Identify and select objects | Store metadata |
| **Usage** | Kubernetes uses them | Kubernetes ignores them |
| **Queried** | Yes (`kubectl get pods -l app=frontend`) | No |
| **Size limit** | 63 characters | 256 KB total |
| **Restrictions** | Strict naming rules | More flexible |
