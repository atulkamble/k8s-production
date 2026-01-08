<div align="center">
<h1>🚀 MyApp</h1>
<p><strong>Built with ❤️ by <a href="https://github.com/atulkamble">Atul Kamble</a></strong></p>

<p>
<a href="https://codespaces.new/atulkamble/template.git">
<img src="https://github.com/codespaces/badge.svg" alt="Open in GitHub Codespaces" />
</a>
<a href="https://vscode.dev/github/atulkamble/template">
<img src="https://img.shields.io/badge/Open%20with-VS%20Code-007ACC?logo=visualstudiocode&style=for-the-badge" alt="Open with VS Code" />
</a>
<a href="https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/atulkamble/template">
<img src="https://img.shields.io/badge/Dev%20Containers-Ready-blue?logo=docker&style=for-the-badge" />
</a>
<a href="https://desktop.github.com/">
<img src="https://img.shields.io/badge/GitHub-Desktop-6f42c1?logo=github&style=for-the-badge" />
</a>
</p>

<p>
<a href="https://github.com/atulkamble">
<img src="https://img.shields.io/badge/GitHub-atulkamble-181717?logo=github&style=flat-square" />
</a>
<a href="https://www.linkedin.com/in/atuljkamble/">
<img src="https://img.shields.io/badge/LinkedIn-atuljkamble-0A66C2?logo=linkedin&style=flat-square" />
</a>
<a href="https://x.com/atul_kamble">
<img src="https://img.shields.io/badge/X-@atul_kamble-000000?logo=x&style=flat-square" />
</a>
</p>

<strong>Version 1.0.0</strong> | <strong>Last Updated:</strong> January 2026
</div>


## 🏗️ Production Kubernetes Reference Architecture

![Image](https://platform9.com/media/kubernetes-constructs-concepts-architecture.jpg)

![Image](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

![Image](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2018/11/20/image1-1.png)

![Image](https://tetrate.io/.netlify/images?h=549\&q=90\&url=_astro%2Fimage-1024x549.Dst0COpw.png\&w=1024)

### 🔹 High-Level Architecture

```
Internet
   |
[ Cloud LB / ALB / NLB ]
   |
[ Ingress Controller ]
   |
[ Services (ClusterIP) ]
   |
[ Pods (Deployments) ]
   |
[ Nodes (VMs / Bare Metal) ]
   |
[ Control Plane (HA) ]
```

### 🔹 Core Components (Production)

| Layer         | Best Practice                |
| ------------- | ---------------------------- |
| Control Plane | 3+ masters (HA)              |
| Worker Nodes  | Auto-scaling groups / VMSS   |
| Networking    | CNI (Calico / Cilium)        |
| Ingress       | NGINX / ALB / Traefik        |
| Storage       | CSI (EBS, Azure Disk, NFS)   |
| Observability | Prometheus + Grafana         |
| Security      | RBAC, PSP/OPA, NetworkPolicy |
| CI/CD         | GitOps (ArgoCD / Flux)       |

---

## 👥 Kubernetes Roles & Responsibilities (Enterprise View)

### 1️⃣ Cluster Administrator

**Responsibilities**

* Cluster creation & upgrades
* RBAC, namespaces
* Network policies
* Storage classes
* Disaster recovery

**Access**

```yaml
verbs: ["*"]
resources: ["*"]
```

---

### 2️⃣ Platform / DevOps Engineer

**Responsibilities**

* CI/CD pipelines
* Helm charts
* Ingress & monitoring
* Secrets management
* Autoscaling (HPA/VPA)

**Access Scope**

```yaml
verbs: ["create","update","get","list"]
resources: ["deployments","services","ingress"]
```

---

### 3️⃣ Application Developer

**Responsibilities**

* Write deployment YAML
* Define resource limits
* Health probes
* ConfigMaps & Secrets usage

❌ No access to:

* Nodes
* RBAC
* Namespaces

---

### 4️⃣ SRE / Operations

**Responsibilities**

* Monitoring & alerting
* Performance tuning
* Incident response
* Scaling & capacity planning

---

### 5️⃣ Security / Compliance Team

**Responsibilities**

* RBAC audits
* Network policy enforcement
* Image scanning
* Secrets rotation

---

## 🔐 Production Security Best Practices

### ✅ RBAC (Least Privilege)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: prod
rules:
- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get","list"]
```

---

### ✅ Network Policies (Zero Trust)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

### ✅ Secrets (Never Hardcode)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlwYXNz
```

---

## ⚙️ Production Deployment Best Practices (YAML)

### ✅ Deployment with Limits, Probes & Replicas

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deploy
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

---

### ✅ Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## 🌐 Ingress – Production Ready

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-svc
            port:
              number: 80
```

---

## 📈 Monitoring & Observability

![Image](https://earthly.dev/blog/assets/images/grafana-and-prometheus-k8s/k8snodes.png)

![Image](https://devopscube.com/content/images/2025/03/image-7-56.png)

![Image](https://www.simform.com/wp-content/uploads/2022/08/Kubernetes-cluster.png)

### Stack

* **Metrics:** Prometheus
* **Dashboards:** Grafana
* **Logs:** EFK / Loki
* **Alerts:** Alertmanager

---

## 🚀 CI/CD & GitOps (Recommended)

### Flow

```
Git Commit
  ↓
CI (Build + Scan)
  ↓
Helm / Kustomize
  ↓
GitOps Tool (ArgoCD)
  ↓
Kubernetes Cluster
```

### Why GitOps?

✔ Versioned infrastructure
✔ Easy rollback
✔ Audit-friendly
✔ Secure

---

## 🧠 Production Checklist (Must-Have)

✔ Multi-AZ cluster
✔ Resource requests & limits
✔ PodDisruptionBudgets
✔ NetworkPolicies
✔ RBAC per role
✔ Health checks
✔ Autoscaling
✔ Monitoring & alerts
✔ Backup (Velero)

---

## 📌 Final Recommendation (Industry Standard)

* **Managed K8s**: AKS / EKS / GKE
* **Ingress**: NGINX or Cloud LB
* **Security**: OPA + Kyverno
* **CI/CD**: GitHub Actions + ArgoCD
* **Monitoring**: Prometheus + Grafana

---
Below is a **complete, production-ready AKS (Azure Kubernetes Service) guide** with **roles & responsibilities, step-by-step setup, and real YAML/CLI/Terraform-ready code**, written the way it’s used in **enterprise Azure environments**.

---

# 🚀 Azure Kubernetes Service (AKS) – Production Guide

**Roles • Responsibilities • Architecture • Steps • Codes • Docs**

---

## 🏗️ AKS Production Architecture (Azure)

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks-microservices/images/microservices-architecture.svg)

![Image](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/aks-agic/media/aks-agic.svg)

![Image](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/images/aks-baseline-architecture.svg)

![Image](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/scenarios/app-platform/aks/media/network-private-cluster.png)

### 🔹 High-Level Architecture

```
Users / Internet
     |
 Azure Load Balancer / Application Gateway
     |
  Ingress Controller (NGINX / AGIC)
     |
  Kubernetes Services (ClusterIP)
     |
  Pods (Deployments)
     |
  Node Pools (VMSS)
     |
  AKS Control Plane (Azure Managed)
```

### 🔹 Azure Components Used

| Layer         | Azure Service                 |
| ------------- | ----------------------------- |
| Control Plane | AKS (Managed)                 |
| Compute       | VM Scale Sets                 |
| Networking    | Azure VNet + Subnets          |
| Ingress       | App Gateway / NGINX           |
| Identity      | Azure AD + RBAC               |
| Secrets       | Azure Key Vault               |
| Monitoring    | Azure Monitor + Prometheus    |
| CI/CD         | GitHub Actions / Azure DevOps |

---

## 👥 AKS Roles & Responsibilities (Enterprise)

---

## 1️⃣ AKS Cluster Administrator (Azure Infra / Cloud Team)

### 🔹 Responsibilities

* Create AKS cluster
* Upgrade Kubernetes version
* Node pools (system / user)
* Azure RBAC & AKS RBAC
* Networking & security
* Backup & DR

### 🔹 Azure Role

```text
Azure Kubernetes Service RBAC Cluster Admin
```

---

## 2️⃣ Platform / DevOps Engineer

### 🔹 Responsibilities

* CI/CD pipelines
* Ingress controller
* Helm charts
* Autoscaling (HPA)
* Monitoring & logging
* Secrets integration (Key Vault)

### 🔹 Kubernetes Access

```yaml
verbs: ["create","update","delete","get","list"]
resources: ["deployments","services","ingress","configmaps"]
```

---

## 3️⃣ Application Developer

### 🔹 Responsibilities

* App Docker image
* Deployment YAML
* ConfigMaps & Secrets usage
* Health probes

### ❌ No Access To

* Nodes
* RBAC
* Network policies

---

## 4️⃣ SRE / Operations

### 🔹 Responsibilities

* Monitoring alerts
* Incident response
* Capacity planning
* Performance tuning

---

## 5️⃣ Security / Compliance Team

### 🔹 Responsibilities

* RBAC audits
* Network policies
* Pod security
* Image vulnerability scanning

---

## ⚙️ Step-by-Step: Create AKS (Azure CLI – Production Ready)

---

### 🔹 Step 1: Login & Set Subscription

```bash
az login
az account set --subscription "<SUBSCRIPTION_ID>"
```

---

### 🔹 Step 2: Create Resource Group

```bash
az group create \
  --name aks-prod-rg \
  --location eastus
```

---

### 🔹 Step 3: Create AKS Cluster (AAD + RBAC)

```bash
az aks create \
  --resource-group aks-prod-rg \
  --name aks-prod \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --enable-managed-identity \
  --enable-aad \
  --enable-azure-rbac \
  --network-plugin azure \
  --enable-addons monitoring \
  --generate-ssh-keys
```

---

### 🔹 Step 4: Connect to Cluster

```bash
az aks get-credentials \
  --resource-group aks-prod-rg \
  --name aks-prod
```

```bash
kubectl get nodes
```

---

## 🔐 AKS RBAC (Kubernetes Level)

### 🔹 Developer Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-role
  namespace: dev
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get","list","create","update"]
```

### 🔹 RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-binding
  namespace: dev
subjects:
- kind: User
  name: dev@company.com
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔐 Network Security (AKS Network Policy)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

## 📦 Application Deployment (Production YAML)

### 🔹 Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: prod
```

---

### 🔹 Deployment (Best Practices)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: prod
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myacr.azurecr.io/web:v1
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        readinessProbe:
          httpGet:
            path: /
            port: 80
```

---

### 🔹 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
  namespace: prod
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

---

## 🌐 Ingress (AKS + NGINX)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: prod
spec:
  rules:
  - host: app.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

---

## 📈 Autoscaling (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: prod
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## 🔍 Monitoring (Azure + Prometheus)

![Image](https://kodekloud.com/kk-media/image/upload/v1752869500/notes-assets/images/Azure-Kubernetes-Service-Container-insights-for-AKS/azure-monitor-diagram-components-processes.jpg)

![Image](https://learn.microsoft.com/en-us/samples/azure-samples/aks-managed-prometheus-and-grafana-bicep/aks-managed-prometheus-and-grafana-bicep/media/architecture.png)

### 🔹 Tools

* Azure Monitor (Container Insights)
* Managed Prometheus
* Grafana
* Log Analytics

---

## 🔄 CI/CD (AKS Best Practice)

### 🔹 Flow

```
GitHub
 → CI (Build + Scan)
 → Push to ACR
 → Helm / YAML
 → ArgoCD
 → AKS
```

---

## ✅ AKS Production Checklist

✔ Azure AD + RBAC
✔ Separate system & user node pools
✔ Resource limits
✔ Network policies
✔ HPA enabled
✔ Ingress secured (TLS)
✔ Monitoring & alerts
✔ Backup (Velero)

---

## 📁 Recommended GitHub Repo Structure

```
aks-production/
├── aks/
│   ├── cluster-create.md
│   ├── rbac.yaml
│   └── network-policy.yaml
├── app/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── hpa.yaml
├── ci-cd/
│   └── github-actions.yaml
└── README.md
```

---

