# Platform Operations (GitOps Repository)

This repository defines the **desired state** of the platform.

It contains:
- Helm charts
- Argo CD Applications
- Kubernetes configuration
- Environment-specific values

It is continuously reconciled by **Argo CD**.

---

## 🎯 Purpose

This repo answers one question:

> “What should be running in the cluster right now?”

Git is the **single source of truth**.

---

## 📂 Repository Structure

platform-ops/ 
      helm/
          api-node/
          python-worker/
          core-go/
      clusters
          dev/
             api-node/
             python-workers/
             core-go/
      argocd/
           api-node.yml
           core-go.yml
           worker-python.yml


---

## ⚙️ Helm Charts

Each service has:
- `Chart.yaml` – metadata
- `values.yaml` – environment configuration
- `templates/` – Kubernetes manifests

Helm is used for:
- reusability
- versioned upgrades
- rollback support

---

## 🔁 GitOps with Argo CD

Argo CD:
- watches this repository
- renders Helm charts
- applies them to Kubernetes
- continuously enforces desired state

### Enabled features
- Automated sync
- Drift correction (self-heal)
- Pruning of deleted resources

No `kubectl apply` is used in production.

---

## 🔄 Image Promotion Flow

1. CI builds images in `platform` repo
2. Images are tagged with Git SHA
3. CI updates `image.tag` in Helm `values.yaml`
4. Changes are committed to this repo
5. Argo CD deploys automatically

Rollback = `git revert`.

---

## 🌐 Networking

- Internal service-to-service via Kubernetes DNS
- External access via F5 NGINX Ingress
- TLS termination at Ingress

---

## 🔐 Security

- Non-root workloads
- Read-only root filesystems
- Least-privilege containers
- No secrets committed to Git

---

## 🧠 Design Philosophy

- GitOps first
- Separation of concerns (CI vs CD)
- Kubernetes-native tooling
- Minimal but production-aligned

---

## 📄 Related Repository

➡️ **Application Source & CI**
- https://github.com/zainasr/platform
