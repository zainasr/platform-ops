Platform Operations (GitOps Repository)

This repository defines the desired state of the platform and is managed using GitOps principles.

It contains:

Helm charts for application workloads

Argo CD Applications

Kubernetes configuration

Environment-specific values

Manual observability setup (Prometheus)

All changes to the cluster are driven only by Git and reconciled by Argo CD.

🎯 Purpose

This repository answers a single question:

“What should be running in the cluster right now?”

Git is the single source of truth.
The cluster is treated as disposable.

📂 Repository Structure
platform-ops/
├── helm/
│   ├── api-node/
│   ├── core-go/
│   └── worker-python/
│
├── argocd/
│   ├── api-node.yml
│   ├── core-go.yml
│   └── worker-python.yml
│
├── observability/
│   └── prometheus/
│       ├── namespace.yaml
│       ├── rbac.yaml
│       ├── configmap.yaml
│       ├── deployment.yaml
│       └── service.yaml
│
└── README.md

⚙️ Helm Charts

Each service chart contains:

Chart.yaml – chart metadata

values.yaml – environment configuration

templates/ – Kubernetes manifests

Helm is used for:

Reusability across environments

Controlled rollouts

Versioned upgrades

Rollback support

All charts are GitOps-safe and environment-agnostic.

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