# Minikube Deployment Demo - Learning Project

A hands-on learning project to master Kubernetes, GitOps, and microservices architecture using Minikube, ArgoCD, and NGINX Ingress.

## 🎯 Project Overview

This is a **GitOps-driven Kubernetes project** featuring 3 microservices managed by ArgoCD using the App-of-Apps pattern:

### Applications
1. **web-app** - Phone store (NGINX, port 80)
2. **token-app** - Node.js API (port 8080)
3. **payment-app** - Node.js + MongoDB (port 3000 + MongoDB)

### Technologies Stack
- **Kubernetes** (Minikube)
- **ArgoCD** (GitOps automation)
- **NGINX Ingress Controller** (Traffic routing)
- **MongoDB** (Database with authentication)
- **TLS/HTTPS** (mkcert for local development)
- **NetworkPolicies** (Zero-trust security)

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Browser                           │
└───────────────────┬──────────────────────────────────┘
                    │ HTTPS
                    ▼
          ┌─────────────────────┐
          │  Ingress (NGINX)    │
          │  - web.apps.local   │
          │  - token.apps.local │
          │  - payment.apps.local│
          └──────────┬──────────┘
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
    ┌────────┐  ┌────────┐  ┌────────┐
    │Web App │  │Token   │  │Payment │
    │(Port   │  │App     │  │App     │
    │80)     │  │(8080)  │  │(3000)  │
    └────────┘  └────────┘  └───┬────┘
                                 │
                                 ▼
                            ┌─────────┐
                            │MongoDB  │
                            │(27017)  │
                            └─────────┘
```

---

## 📚 Learning Path (Step-by-Step)

### **Phase 1: Setup Foundation** ⚙️ (30 minutes)
**Goal:** Get your local Kubernetes environment ready

1. Install Minikube
   ```bash
   # Linux
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   minikube start
   ```

2. Install ArgoCD
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   kubectl get pods -n argocd
   ```

3. Access ArgoCD UI
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:80
   # Get password:
   kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
   ```

4. Install NGINX Ingress Controller
   ```bash
   minikube addons enable ingress
   # OR
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

5. Configure local DNS
   ```bash
   # Add to /etc/hosts (Linux) or C:\Windows\System32\drivers\etc\hosts (Windows)
   127.0.0.1   web.apps.local token.apps.local payment.apps.local
   ```

---

### **Phase 2: Deploy Web App** 🌐 (1-2 hours)
**Goal:** Understand basic Kubernetes resources

**Create these files:**
- `apps/dev/web-app/deployment.yaml` - Pod specification
- `apps/dev/web-app/service.yaml` - Internal networking
- `apps/dev/web-app/ingress.yaml` - External access
- `apps/dev/web-app/application.yaml` - ArgoCD config

**Key Concepts to Learn:**
- Container images and ports
- Labels and selectors
- Service types (ClusterIP)
- Ingress rules and hosts
- ArgoCD Application CRD

**Test:** Visit `http://web.apps.local`

---

### **Phase 3: Deploy Token App** 🔑 (1-2 hours)
**Goal:** Master port mapping and service configuration

**Create same structure as web-app but learn:**
- Different container ports (8080)
- `targetPort` vs `port` in Services
- `ingressClassName: nginx`
- Port-forward for debugging

**Key Debugging Commands:**
```bash
kubectl get pods -n token-app
kubectl logs <pod-name> -n token-app
kubectl describe pod <pod-name> -n token-app
kubectl port-forward svc/token-app -n token-app 8080:80
```

**Test:** Visit `http://token.apps.local`

---

### **Phase 4: Deploy Payment App** 💳 (2-3 hours)
**Goal:** Learn database integration and secrets management

**Create these files:**
- `apps/dev/payment-app/mongo-deployment.yaml` - Database pod
- `apps/dev/payment-app/mongo-service.yaml` - DB networking
- `apps/dev/payment-app/secret.yaml` - Credentials (base64 encoded)
- `apps/dev/payment-app/configmap.yaml` - Non-sensitive config
- `apps/dev/payment-app/web-deployment.yaml` - App pod
- `apps/dev/payment-app/web-service.yaml` - App networking
- `apps/dev/payment-app/web-ingress.yaml` - External access
- `apps/dev/payment-app/application.yaml` - ArgoCD config

**Critical Concepts:**
- Secrets vs ConfigMaps
- Base64 encoding for secrets
- Environment variable injection
- MongoDB connection strings
- Service DNS in Kubernetes (`mongodb://service-name:27017`)
- Pod-to-pod communication

**Test:** Visit `http://payment.apps.local`

---

### **Phase 5: ArgoCD App-of-Apps** 🚀 (1 hour)
**Goal:** Automate deployments with GitOps

**Create:**
- `bootstrap/root-app.yaml` - Parent ArgoCD application

**Key Configuration:**
```yaml
source:
  repoURL: https://github.com/YOUR-USERNAME/minikube-deployment-demo.git
  path: apps/dev
  directory:
    recurse: true  # 👈 Critical for subdirectories
```

**Deploy:**
```bash
kubectl apply -f bootstrap/root-app.yaml
```

**Learn:**
- GitOps principles
- Declarative deployments
- Auto-sync and self-healing
- ArgoCD Application CRD
- App-of-Apps pattern

---

### **Phase 6: Security & Production Features** 🔒 (2-3 hours)
**Goal:** Production-grade security and reliability

**Implement:**

1. **HTTPS/TLS with mkcert**
   ```bash
   mkcert -install
   mkcert web.apps.local token.apps.local payment.apps.local
   kubectl create secret tls apps-local-tls \
     --cert=cert.pem --key=key.pem -n ingress-nginx
   ```

2. **NetworkPolicies** (Zero-trust security)
   - Default deny all traffic
   - Allow ingress from NGINX only
   - Allow DNS for all pods
   - Block lateral pod-to-pod movement

3. **Resource Limits**
   ```yaml
   resources:
     requests:
       memory: "128Mi"
       cpu: "100m"
     limits:
       memory: "256Mi"
       cpu: "200m"
   ```

4. **Persistent Volumes for MongoDB**
   - PersistentVolumeClaim
   - Data survives pod restarts

---

## 🔑 Critical Concepts to Master

### Port Mapping Chain
```
Browser → Ingress (80) → Service (port: 80) → Pod (targetPort: 8080)
```

### Service Selectors
```yaml
# Deployment
labels:
  app: token-app  # 👈 Must match

# Service
selector:
  app: token-app  # 👈 Must match
```

### MongoDB Connection in Kubernetes
```yaml
# ConfigMap
DB_URL: mongodb://payment-mongo:27017  # 👈 Use Service name

# Secret (base64 encoded)
MONGO_USER: dXNlcg==
MONGO_PASSWORD: cGFzc3dvcmQ=
```

### Ingress Requirements
```yaml
spec:
  ingressClassName: nginx  # 👈 Required for NGINX
  rules:
    - host: app.apps.local  # 👈 Must match /etc/hosts
```

### ArgoCD App-of-Apps
```yaml
directory:
  recurse: true  # 👈 Required to scan subdirectories
```

---

## 📁 Project Structure

```
minikube-deployment-demo/
├── README.md
├── bootstrap/
│   └── root-app.yaml              # ArgoCD App-of-Apps
└── apps/
    └── dev/
        ├── web-app/
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   ├── ingress.yaml
        │   └── application.yaml
        ├── token-app/
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   ├── ingress.yaml
        │   └── application.yaml
        └── payment-app/
            ├── mongo-deployment.yaml
            ├── mongo-service.yaml
            ├── web-deployment.yaml
            ├── web-service.yaml
            ├── web-ingress.yaml
            ├── configmap.yaml
            ├── secret.yaml
            └── application.yaml
```

---

## 🐛 Common Issues & Solutions

### 503 Service Unavailable
**Cause:** Service selector doesn't match pod labels  
**Fix:** Ensure labels match exactly
```bash
kubectl get endpoints -n <namespace>  # Should show IPs
```

### Ingress 404 Not Found
**Cause:** Missing `ingressClassName: nginx`  
**Fix:** Add to ingress spec

### MongoDB Connection Timeout
**Cause:** Wrong service hostname  
**Fix:** Use Kubernetes service name: `mongodb://service-name:27017`

### ArgoCD Shows Empty
**Cause:** Missing `directory.recurse: true`  
**Fix:** Add to root-app.yaml source spec

### Pods Not Starting
**Cause:** Image not loaded in Minikube  
**Fix:**
```bash
minikube image load <image-name>
```

---

## 🛠️ Useful Commands

### Minikube
```bash
minikube start
minikube status
minikube dashboard
minikube tunnel  # For LoadBalancer services
```

### Kubernetes Debugging
```bash
# Pods
kubectl get pods -A
kubectl describe pod <name> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl exec -it <pod> -n <namespace> -- sh

# Services & Endpoints
kubectl get svc -A
kubectl get endpoints -n <namespace>

# Ingress
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>

# Port Forward (bypass ingress)
kubectl port-forward svc/<service> -n <namespace> 8080:80
```

### ArgoCD
```bash
# Get password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode

# Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:80

# List apps
kubectl get applications -n argocd

# Sync manually
kubectl argocd app sync <app-name>
```

---

## 📖 Learning Resources

- **Kubernetes Docs:** https://kubernetes.io/docs/
- **ArgoCD Docs:** https://argo-cd.readthedocs.io/
- **NGINX Ingress:** https://kubernetes.github.io/ingress-nginx/
- **Minikube:** https://minikube.sigs.k8s.io/

---

## 🎓 What You'll Learn

By completing this project, you will understand:

✅ Kubernetes core resources (Pods, Services, Deployments, Ingress)  
✅ ConfigMaps and Secrets management  
✅ Service discovery and DNS in Kubernetes  
✅ Ingress controllers and routing  
✅ GitOps with ArgoCD  
✅ App-of-Apps pattern  
✅ Database integration in K8s  
✅ Security with NetworkPolicies  
✅ TLS/HTTPS configuration  
✅ Debugging Kubernetes applications  
✅ Production-ready patterns  

---

## 🚀 Next Steps

1. **Clone this repo** and start with Phase 1
2. **Create each file manually** - don't copy/paste everything
3. **Test after each phase** - make sure it works before moving forward
4. **Read error messages** - they teach you a lot
5. **Experiment** - break things and fix them
6. **Document your learnings** - add notes as you go

---

## 📝 License

This is a learning project. Feel free to use and modify as needed.

---

**Happy Learning! 🎉**
