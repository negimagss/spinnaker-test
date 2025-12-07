# 🚀 Spinnaker Getting Started Guide

## ✅ Your Spinnaker is Running!

All services are healthy and operational. Here's how to use it.

---

## 📋 Required Port Forwards

Spinnaker needs **TWO** port-forwards to work properly:

### 1. Port-forward Deck (UI)
```bash
kubectl port-forward -n spinnaker-operator svc/spin-deck 9000:9000
```

### 2. Port-forward Gate (API)
```bash
kubectl port-forward -n spinnaker-operator svc/spin-gate 8084:8084
```

> **⚠️ IMPORTANT**: You need **BOTH** port-forwards running in separate terminal windows!

---

## 🌐 Access Spinnaker

Once both port-forwards are running, open your browser to:

**http://localhost:9000**

---

## 🎯 Create Your First Application

### Option 1: Via UI

1. Go to http://localhost:9000/#/applications
2. Click **"Actions"** → **"Create Application"**
3. Fill in the form:
   - **Name**: `hello-app`
   - **Owner Email**: `your-email@example.com`
   - **Description**: (optional)
4. Click **"Create"**

### Option 2: Via API (curl)

```bash
curl -X POST http://localhost:8084/applications \
  -H "Content-Type: application/json" \
  -d '{
    "name": "hello-app",
    "email": "test@example.com",
    "description": "My first Spinnaker application"
  }'
```

---

## 🔧 Create Your First Pipeline

### Option 1: Simple Manual Pipeline

1. Go to your application: http://localhost:9000/#/applications/hello-app
2. Click **"Pipelines"** tab
3. Click **"Configure a new pipeline"**
4. Enter pipeline name: `hello-pipeline`
5. Click **"Create"**
6. Add a stage:
   - Click **"Add stage"**
   - Select **"Wait"** (simplest stage type)
   - Set wait time: `30` seconds
7. Click **"Save Changes"**

### Option 2: Import Pipeline from JSON

You already have a pipeline template at:
`/Users/shardulnegi/Downloads/Neology/EKS/spinnaker/import.json`

To import it:
1. Create the application first (see above)
2. Use the Spinnaker API:

```bash
curl -X POST http://localhost:8084/pipelines \
  -H "Content-Type: application/json" \
  -d @/Users/shardulnegi/Downloads/Neology/EKS/spinnaker/import.json
```

---

## 🐛 Troubleshooting

### "Error initializing dialog" or "Gate endpoint not accessible"

**Problem**: The UI can't connect to the API.

**Solution**: Make sure **BOTH** port-forwards are running:
```bash
# Terminal 1
kubectl port-forward -n spinnaker-operator svc/spin-deck 9000:9000

# Terminal 2
kubectl port-forward -n spinnaker-operator svc/spin-gate 8084:8084
```

### "Error fetching pipeline templates"

**This is normal!** A fresh Spinnaker installation has no pipeline templates. You can:
- Ignore this message
- Create pipelines directly (not from templates)
- Create your own templates later

### Check Service Health

```bash
# Check all pods are running
kubectl get pods -n spinnaker-operator

# Check Gate health
curl http://localhost:8084/health

# Check Deck is accessible
curl http://localhost:9000
```

---

## 📊 Verify Everything is Working

Run this command to see all services:

```bash
kubectl get pods -n spinnaker-operator
```

You should see all pods in `Running` status:
- ✅ spin-clouddriver
- ✅ spin-deck (UI)
- ✅ spin-echo
- ✅ spin-front50 (persistence)
- ✅ spin-gate (API)
- ✅ spin-orca (orchestration)
- ✅ spin-redis
- ✅ spin-rosco

---

## 🎓 Next Steps for Learning

Now that Spinnaker is running, you can:

1. **Create an application** (see above)
2. **Create a simple pipeline** with a Wait stage
3. **Run the pipeline** manually
4. **Explore the complexity**:
   - Notice how many clicks it takes to do simple things
   - See how pipelines are defined (JSON-heavy)
   - Compare this to ArgoCD's GitOps approach

### Why Companies Leave Spinnaker

As you use it, you'll notice:
- 🔴 **Complex UI** - Many nested menus and options
- 🔴 **Heavy setup** - 8+ microservices just to run
- 🔴 **Pipeline as code is awkward** - JSON-based, not GitOps
- 🔴 **Steep learning curve** - Lots of concepts to learn

Compare this to **ArgoCD**:
- ✅ Single deployment
- ✅ GitOps-native (everything in Git)
- ✅ Simpler UI
- ✅ Kubernetes-native

---

## 🧹 Cleanup

When you're done learning, clean up with:

```bash
# Delete Spinnaker
kubectl delete spinnakerservice spinnaker -n spinnaker-operator

# Delete MinIO
kubectl delete namespace minio

# Delete the operator
kubectl delete namespace spinnaker-operator

# Delete the kind cluster (if you want to remove everything)
kind delete cluster --name spinnaker
```

---

## 📚 Additional Resources

- [Spinnaker Documentation](https://spinnaker.io/docs/)
- [Pipeline Expressions Guide](https://spinnaker.io/docs/guides/user/pipeline/expressions/)
- [Spinnaker vs ArgoCD Comparison](https://www.armosec.io/blog/spinnaker-vs-argo-cd/)

---

**Happy Learning! 🎉**
