# Platform Operations - GitOps Repository

[![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-orange)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-blue)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-v3-0f1689)](https://helm.sh/)
[![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-e6522c)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Dashboards-Grafana-f46800)](https://grafana.com/)

> **Production-ready Kubernetes infrastructure managed through GitOps principles with comprehensive observability**

This repository serves as the **single source of truth** for the entire platform infrastructure. It defines the desired state of all Kubernetes resources and is automatically reconciled by Argo CD. All changes to the cluster are driven exclusively through Git commits—no manual `kubectl apply` commands.

---

## 📋 Table of Contents

- [Philosophy & Purpose](#-philosophy--purpose)
- [Repository Structure](#-repository-structure)
- [Technology Stack](#-technology-stack)
- [Helm Charts](#️-helm-charts)
- [GitOps with Argo CD](#-gitops-with-argo-cd)
- [Observability Stack](#-observability-stack)
- [Local Setup Guide](#-local-setup-guide)
- [CI/CD Integration](#-cicd-integration)
- [Security & Best Practices](#-security--best-practices)
- [Troubleshooting](#-troubleshooting)
- [Cleanup & Reset](#-cleanup--reset)

---

## 🎯 Philosophy & Purpose

### Core Question

**"What should be running in the cluster right now?"**

This repository provides the definitive answer through declarative configuration.

### GitOps Principles

✅ **Git as Single Source of Truth**  
✅ **Declarative Infrastructure**  
✅ **Automated Reconciliation**  
✅ **Immutable Deployments**  
✅ **Auditable Change History**  
✅ **Self-Healing Capabilities**  

### Design Principles

- **Separation of Concerns**: Application code lives in [`platform`](https://github.com/zainasr/platform) repository
- **Environment Parity**: Same patterns work across dev, staging, and production
- **Disposable Infrastructure**: Clusters can be recreated from scratch in minutes
- **Progressive Delivery**: Controlled rollouts with automated rollback capabilities
- **Observability First**: Built-in monitoring, metrics, and visualization

---

## 📂 Repository Structure

```
platform-ops/
│
├── helm/                          # Helm charts for all services
│   ├── api-node/
│   │   ├── Chart.yaml            # Chart metadata
│   │   ├── values.yaml           # Default configuration
│   │   └── templates/            # Kubernetes manifests
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── ingress.yaml
│   │       ├── hpa.yaml          # Horizontal Pod Autoscaler
│   │       ├── pdb.yaml          # Pod Disruption Budget
│   │       └── servicemonitor.yaml
│   │
│   ├── core-go/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       └── ...
│   │
│   └── worker-python/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           └── ...
│
├── argocd/                        # Argo CD Application definitions
│   ├── api-node.yaml             # Application CRD for api-node
│   ├── core-go.yaml              # Application CRD for core-go
│   └── worker-python.yaml        # Application CRD for worker-python
│
├── observability/                 # Monitoring and observability stack
│   ├── prometheus/               # Prometheus metrics collection
│   │   ├── namespace.yaml        # Dedicated namespace
│   │   ├── rbac.yaml             # ServiceAccount + ClusterRole
│   │   ├── configMap.yml         # Scrape configs & rules
│   │   ├── deployment.yaml       # Prometheus deployment
│   │   └── service.yaml          # Prometheus service
│   │
│   └── grafana/                  # Grafana dashboards
│       ├── deployment.yaml       # Grafana deployment
│       ├── service.yaml          # Grafana service
│       ├── configmap-dashboards.yaml
│       └── configmap-datasources.yaml
│
├── ingress/                       # Ingress controller (NGINX)
│   └── ingress-nginx.yaml
│
└── README.md                      # This file
```

---

## 🛠️ Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Orchestration** | Kubernetes | 1.28+ | Container orchestration |
| **GitOps** | Argo CD | 2.9+ | Continuous deployment |
| **Package Manager** | Helm | 3.13+ | Templating & versioning |
| **Ingress** | NGINX Ingress | 1.9+ | External traffic routing |
| **Metrics** | Prometheus | 2.48+ | Metrics collection |
| **Visualization** | Grafana | 10.2+ | Dashboards & alerting |
| **Container Registry** | GHCR | Latest | Image storage |
| **Local Cluster** | Kind | 0.20+ | Local K8s cluster |

---

## ⚙️ Helm Charts

### Chart Structure

Each service has a dedicated Helm chart following best practices:

```yaml
helm/service-name/
├── Chart.yaml              # Chart metadata & versioning
├── values.yaml            # Default values (overridable)
├── values-dev.yaml        # Development overrides
├── values-prod.yaml       # Production overrides
└── templates/
    ├── deployment.yaml    # Main workload
    ├── service.yaml       # Service definition
    ├── ingress.yaml       # External routing
    ├── hpa.yaml          # Autoscaling config
    ├── pdb.yaml          # Disruption budget
    ├── configmap.yaml    # Configuration data
    ├── secret.yaml       # Sensitive data (sealed)
    └── servicemonitor.yaml # Prometheus scraping
```

### Chart Components

#### 1. **Deployment**
- Container specifications
- Resource limits and requests
- Security context (non-root)
- Health probes (liveness/readiness)
- Environment variables
- Volume mounts

#### 2. **Service**
- ClusterIP service type
- Port definitions
- Selector labels
- Session affinity (if needed)

#### 3. **Ingress**
- Path-based routing
- TLS termination
- SSL certificates
- Rate limiting annotations

#### 4. **Horizontal Pod Autoscaler (HPA)**
- CPU-based scaling
- Memory-based scaling
- Custom metrics (optional)

#### 5. **Pod Disruption Budget (PDB)**
- Minimum available pods
- Maximum unavailable pods
- High availability guarantees

### Key Benefits

✅ **Reusability**: Same chart across dev/staging/prod  
✅ **Versioning**: Semantic versioning for charts  
✅ **Templating**: DRY principle with Go templates  
✅ **Rollback**: Built-in rollback capabilities  
✅ **Testing**: `helm template` and `helm lint`  
✅ **GitOps Compatible**: Declarative and deterministic  

### Example Values (api-node)

```yaml
# values.yaml
replicaCount: 3

image:
  repository: ghcr.io/org/api-node
  tag: "a1b2c3d"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 3000

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

🔁 GitOps with Argo CD

Argo CD is responsible for continuous reconciliation.

Responsibilities

Watch this repository

Render Helm charts

Apply Kubernetes manifests

Enforce desired state continuously

Enabled Features

Automated sync

Drift correction (self-heal)

Resource pruning

❌ No kubectl apply is used for application workloads.

🔄 CI → GitOps Image Promotion Flow

This repository does not build images.

Flow

CI builds images in the platform repository

Images are tagged with Git commit SHA

CI updates image.repository and image.tag in Helm values.yaml

Changes are committed to this repository

Argo CD detects the change and deploys automatically

Rollback = Git revert

🌐 Networking

Service-to-service communication via Kubernetes DNS

External access via NGINX Ingress

TLS termination at Ingress (local / self-signed for Kind)

🔐 Security Posture

Containers run as non-root

Read-only root filesystems

Least-privilege security context

No secrets committed to Git

Registry access via imagePullSecrets

📊 Observability (Manual Prometheus Setup)

Observability is intentionally installed manually (no Helm) to deeply understand Prometheus internals.

What is monitored

Application metrics (/metrics)

HTTP request counters and latency

Background worker metrics

What is NOT included (yet)

node-exporter

kube-state-metrics

Alertmanager

🔍 Prometheus Architecture (Manual)

Prometheus is deployed as a Deployment and discovers applications using:

Kubernetes API

kubernetes_sd_configs

Label-based relabeling

Prometheus pulls metrics from:

api-node

core-go

worker-python

Each application exposes /metrics.

📂 Prometheus Manifests

Located at:

observability/prometheus/


Includes:

Namespace

ServiceAccount

RBAC

ConfigMap (prometheus.yml)

Deployment

Service

Prometheus runs in the observability namespace and scrapes apps in default.

🚀 Running the Platform Locally (Clean Start)
1️⃣ Prerequisites

Docker

Kind

kubectl

GitHub account (GHCR access)

2️⃣ Create a Kind Cluster
kind create cluster --name kind-cluster-1
kubectl config use-context kind-kind-cluster-1

3️⃣ Install Argo CD (Fresh)
kubectl create namespace argocd

kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Verify:

kubectl get pods -n argocd


All pods must be Running, especially argocd-repo-server.

4️⃣ Configure GHCR Access (Required)

Create image pull secret:

kubectl create secret docker-registry ghcr-cred \
  --docker-server=ghcr.io \
  --docker-username=<github-username> \
  --docker-password=<github-token-with-read-packages> \
  --docker-email=unused@example.com \
  -n default


Attach to default ServiceAccount:

kubectl patch serviceaccount default \
  -n default \
  -p '{"imagePullSecrets":[{"name":"ghcr-cred"}]}'

5️⃣ Register Applications in Argo CD
kubectl apply -n argocd -f argocd/


Verify:

kubectl get applications -n argocd


Apps should move to:

Synced

Healthy

6️⃣ Install Prometheus (Manual)
kubectl apply -f observability/prometheus/


Access Prometheus:

kubectl port-forward -n observability svc/prometheus 9090:9090


Open:

http://localhost:9090


Check:

Status → Targets

All app targets should be UP

🧹 Cleaning the Cluster (Reset Everything)
Remove Applications
kubectl delete applications -n argocd --all

Remove Workloads
kubectl delete namespace default
kubectl create namespace default

Remove Prometheus
kubectl delete namespace observability

Remove Argo CD (Full Reset)
kubectl delete namespace argocd

🧠 Design Philosophy

GitOps-first

Immutable images

CI ≠ CD separation

Kubernetes-native patterns

Minimal but production-aligned

Learning-focused observability (manual first, Helm later)

📄 Related Repository
Application Source & CI

➡️ https://github.com/zainasr/platform