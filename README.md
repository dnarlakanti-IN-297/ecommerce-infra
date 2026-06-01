# E-Commerce Infrastructure - Helm Charts & K8s Manifests

Infrastructure as Code for the 3-tier E-Commerce application. Contains Helm charts, Kubernetes manifests, and ArgoCD configurations.

## 📁 Repository Structure

```
ecommerce-infra/
├── helm/
│   └── ecommerce/
│       ├── Chart.yaml              # Helm chart metadata
│       ├── values.yaml             # Default values
│       ├── values-dev.yaml         # Dev environment overrides
│       ├── values-prod.yaml        # Production overrides
│       └── templates/              # K8s resource templates
│           ├── frontend-deployment.yaml
│           ├── frontend-service.yaml
│           ├── backend-deployment.yaml
│           ├── backend-service.yaml
│           ├── database-statefulset.yaml
│           ├── database-service.yaml
│           └── ingress.yaml
├── argocd/
│   ├── application-dev.yaml        # ArgoCD app for dev
│   └── application-prod.yaml       # ArgoCD app for prod
└── k8s/
    └── namespaces.yaml             # Namespace definitions
```

## 🚀 Deployment

### Prerequisites

- Kubernetes cluster (GKE, EKS, AKS, or Kind)
- kubectl configured
- Helm 3.x installed
- ArgoCD installed (optional, for GitOps)

### Manual Deployment with Helm

```bash
# Create namespace
kubectl create namespace ecommerce-dev

# Install with default values
helm install ecommerce ./helm/ecommerce -n ecommerce-dev

# Install with environment-specific values
helm install ecommerce ./helm/ecommerce -f helm/ecommerce/values-dev.yaml -n ecommerce-dev

# Upgrade existing deployment
helm upgrade ecommerce ./helm/ecommerce -n ecommerce-dev
```

### GitOps Deployment with ArgoCD

```bash
# Apply ArgoCD application
kubectl apply -f argocd/application-dev.yaml

# ArgoCD will automatically:
# 1. Watch this Git repository
# 2. Detect changes to Helm charts
# 3. Sync changes to the cluster
```

## 🌐 Environments

### Development (ecommerce-dev)
- **Namespace**: `ecommerce-dev`
- **Replicas**: Frontend: 1, Backend: 2
- **Resources**: Low (cost-optimized)
- **Auto-Deploy**: Yes
- **URL**: dev.ecommerce.example.com

### Production (ecommerce-prod)
- **Namespace**: `ecommerce-prod`
- **Replicas**: Frontend: 3, Backend: 5
- **Resources**: High (performance-optimized)
- **Auto-Deploy**: No (manual approval)
- **URL**: www.ecommerce.example.com

## 📊 Helm Chart Structure

### Frontend Deployment
- **Image**: `dnarlakanti/ecommerce-frontend`
- **Port**: 8080
- **Replicas**: Configurable (default: 3)
- **Health Checks**: Liveness and Readiness probes

### Backend Deployment
- **Image**: `dnarlakanti/ecommerce-backend`
- **Port**: 5000
- **Replicas**: Configurable (default: 5)
- **Health Checks**: Liveness and Readiness probes
- **HPA**: Auto-scaling based on CPU (70%)

### Database StatefulSet
- **Image**: `postgres:15-alpine`
- **Port**: 5432
- **Storage**: Persistent Volume (10Gi default)
- **Replicas**: 1 (can add read replicas)

## 🔄 CI/CD Integration

### Jenkins Pipeline Flow

1. **Jenkins** builds and pushes Docker images
2. **Jenkins** updates `values.yaml` with new image tags
3. **Jenkins** commits and pushes to this repo
4. **ArgoCD** detects Git changes
5. **ArgoCD** syncs to Kubernetes cluster

### Image Tag Update

Jenkins updates image tags in `values.yaml`:

```yaml
frontend:
  image:
    tag: abc123  # Git commit SHA
backend:
  image:
    tag: abc123  # Git commit SHA
```

## 🛠️ Customization

### Override Values

Create environment-specific values files:

```yaml
# values-staging.yaml
frontend:
  replicaCount: 2
backend:
  replicaCount: 3
database:
  persistence:
    size: 15Gi
```

Deploy with overrides:

```bash
helm install ecommerce ./helm/ecommerce -f values-staging.yaml -n ecommerce-staging
```

### Update Ingress Hostname

Edit `values.yaml`:

```yaml
ingress:
  hosts:
    - host: your-domain.com
```

## 📝 ArgoCD Configuration

### Application Spec

```yaml
source:
  repoURL: https://github.com/dnarlakanti-IN-297/ecommerce-infra.git
  targetRevision: main
  path: helm/ecommerce

syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Sync Behavior

- **Dev**: Auto-sync enabled (immediate deployment)
- **Prod**: Manual sync (approval required)

## 🔐 Secrets Management

For production, use external secret management:

- **Kubernetes Secrets**: For sensitive data
- **External Secrets Operator**: Sync from Vault/AWS Secrets Manager
- **Sealed Secrets**: Encrypted secrets in Git

## 📦 Helm Commands

```bash
# List installed releases
helm list -n ecommerce-dev

# Get values
helm get values ecommerce -n ecommerce-dev

# Rollback to previous version
helm rollback ecommerce 1 -n ecommerce-dev

# Uninstall
helm uninstall ecommerce -n ecommerce-dev

# Dry-run (test before apply)
helm install ecommerce ./helm/ecommerce --dry-run --debug
```

## 🔗 Related Repositories

- [ecommerce-app](https://github.com/dnarlakanti-IN-297/ecommerce-app) - Application source code

## 👥 Maintainers

- dnarlakanti@cloudbees.com
