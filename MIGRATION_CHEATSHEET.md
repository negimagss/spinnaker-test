# Spinnaker → ArgoCD Migration Cheat Sheet

## Quick Comparison

| What | Spinnaker | ArgoCD |
|------|-----------|--------|
| **Pipeline Definition** | JSON in Spinnaker DB | YAML in Git |
| **Deployment Trigger** | Webhook to Spinnaker | Git commit |
| **Manual Approval** | `manualJudgment` stage | `syncPolicy.automated: false` |
| **Deployment** | Spinnaker pushes to K8s | ArgoCD pulls from Git |
| **UI** | http://spinnaker:9000 | http://argocd-server |
| **CLI** | `spin` | `argocd` |

---

## Common Spinnaker Stages → ArgoCD

### 1. Deploy Stage
**Spinnaker:**
```json
{
  "type": "deployManifest",
  "manifest": {
    "kind": "Deployment",
    "spec": {...}
  }
}
```

**ArgoCD:**
```yaml
# Just put the manifest in Git!
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
spec: {...}
```

### 2. Manual Approval
**Spinnaker:**
```json
{
  "type": "manualJudgment",
  "comments": "Approve to deploy"
}
```

**ArgoCD:**
```yaml
syncPolicy:
  automated: false  # Requires manual sync via UI/CLI
```

### 3. Run Job (Tests)
**Spinnaker:**
```json
{
  "type": "runJob",
  "manifest": {
    "kind": "Job",
    "spec": {...}
  }
}
```

**ArgoCD:**
```yaml
# Use PreSync hook
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
spec: {...}
```

### 4. Notifications
**Spinnaker:**
```json
{
  "notifications": [{
    "type": "slack",
    "channel": "#deployments"
  }]
}
```

**ArgoCD:**
```yaml
# argocd-notifications ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  service.slack: |
    token: $slack-token
  template.app-deployed: |
    message: App {{.app.metadata.name}} deployed
```

---

## Migration Steps

### Step 1: Export Spinnaker Pipeline
```bash
# Get pipeline JSON
curl http://spinnaker-gate:8084/applications/my-app/pipelineConfigs/my-pipeline > pipeline.json

# View it
cat pipeline.json | jq .
```

### Step 2: Extract Kubernetes Manifests
Look for `deployManifest` stages in the JSON:
```json
{
  "type": "deployManifest",
  "manifest": {
    // This is your K8s manifest - copy it!
  }
}
```

### Step 3: Create Git Repository
```bash
mkdir -p k8s/my-app
cd k8s/my-app

# Put manifests here
vim deployment.yaml
vim service.yaml
```

### Step 4: Create ArgoCD Application
```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/mycompany/k8s-manifests
    targetRevision: main
    path: k8s/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Step 5: Deploy ArgoCD Application
```bash
kubectl apply -f argocd-app.yaml

# Watch it sync
argocd app get my-app
argocd app sync my-app
```

---

## Useful Commands

### Spinnaker
```bash
# List applications
curl http://spinnaker-gate:8084/applications

# Get pipeline configs
curl http://spinnaker-gate:8084/applications/APP_NAME/pipelineConfigs

# Get pipeline executions
curl http://spinnaker-gate:8084/applications/APP_NAME/pipelines

# Trigger pipeline
curl -X POST http://spinnaker-gate:8084/pipelines/APP_NAME/PIPELINE_NAME
```

### ArgoCD
```bash
# List applications
argocd app list

# Get app details
argocd app get my-app

# Sync (deploy)
argocd app sync my-app

# View diff
argocd app diff my-app

# Rollback
argocd app rollback my-app
```

---

## Key Differences to Remember

### 1. **Push vs Pull**
- **Spinnaker:** Pushes changes to Kubernetes
- **ArgoCD:** Pulls from Git and syncs to Kubernetes

### 2. **Source of Truth**
- **Spinnaker:** Pipeline JSON in Spinnaker database
- **ArgoCD:** YAML files in Git repository

### 3. **Deployment Trigger**
- **Spinnaker:** Webhook triggers pipeline
- **ArgoCD:** Git commit triggers sync (or manual sync)

### 4. **Rollback**
- **Spinnaker:** Re-run old pipeline version
- **ArgoCD:** `git revert` or sync to previous commit

---

## Common Gotchas

### 1. **Image Tags**
**Spinnaker:** Can use `${trigger.tag}` to get Docker tag
**ArgoCD:** Need to update image tag in Git (use ArgoCD Image Updater)

### 2. **Secrets**
**Spinnaker:** Can reference secrets from various sources
**ArgoCD:** Secrets must be in Git (encrypted with Sealed Secrets or External Secrets)

### 3. **Multi-Environment**
**Spinnaker:** One pipeline with multiple deploy stages
**ArgoCD:** One Application per environment, or use ApplicationSet

### 4. **Manual Approvals**
**Spinnaker:** Built-in `manualJudgment` stage
**ArgoCD:** Disable auto-sync, approve via UI/CLI

---

## Testing Your Migration

### Checklist
- [ ] Spinnaker pipeline deploys successfully
- [ ] Extract all K8s manifests to Git
- [ ] Create ArgoCD Application
- [ ] ArgoCD syncs and deploys same resources
- [ ] Verify both deployments are identical
- [ ] Update CI to commit to Git instead of triggering Spinnaker
- [ ] Test rollback in ArgoCD
- [ ] Decommission Spinnaker pipeline

---

## Resources

- [ArgoCD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Spinnaker API Docs](https://spinnaker.io/docs/reference/api/)
- [GitOps Principles](https://opengitops.dev/)

---

**Good luck with your migration next week!** 🚀
