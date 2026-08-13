# ArgoCD Cheatsheet

Complete reference guide for GitOps continuous deployment with Kubernetes.

## 🎯 ArgoCD Fundamentals

### Installation & Setup

Install ArgoCD:
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd

# Wait for all pods ready
kubectl rollout status deployment/argocd-server -n argocd

# Install a specific version
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.8.0/manifests/install.yaml
```

Access ArgoCD UI:
```bash
# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Access at: https://localhost:8080

# Or expose as LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"LoadBalancer"}}'

# Or use NodePort
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"NodePort"}}'

# Get the LoadBalancer IP
kubectl get svc argocd-server -n argocd
```

Initial Admin Password:
```bash
# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Login with username: admin and the password above

# Change admin password
argocd account update-password \
  --account admin \
  --current-password <current-password> \
  --new-password <new-password>

# Disable admin account (not recommended)
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"admin.enabled":"false"}}'
```

### ArgoCD CLI Installation

Install CLI:
```bash
# macOS
brew install argocd

# Linux
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

# Windows (PowerShell)
$ProgressPreference = 'SilentlyContinue'; iwr -Uri https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe -OutFile argocd.exe

# Verify installation
argocd version

# Login to ArgoCD
argocd login <server> --username admin --password <password>

# Login with insecure flag (self-signed certs)
argocd login <server> --insecure --username admin --password <password>
```

## 📱 Application Management

### Create Applications

Create Application via CLI:
```bash
# Basic application
argocd app create my-app \
  --repo https://github.com/user/repo.git \
  --path ./kubernetes \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# With Helm
argocd app create my-helm-app \
  --repo https://github.com/user/repo.git \
  --path ./helm-chart \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set image.tag=v1.0.0

# With Kustomize
argocd app create my-kustomize-app \
  --repo https://github.com/user/repo.git \
  --path ./kustomize \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Create Application via YAML:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
    path: kubernetes
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```
```bash
# Apply the application
kubectl apply -f application.yaml
```

Application Operations:
```bash
# List applications
argocd app list

# Get application details
argocd app get my-app

# Show application status
argocd app wait my-app

# Sync application
argocd app sync my-app

# Hard sync (ignore differences)
argocd app sync my-app --force

# Sync specific resource
argocd app sync my-app --resource deployment/my-deployment

# Show diff between live and desired state
argocd app diff my-app

# Delete application
argocd app delete my-app
```

### Application Status & Monitoring

Check Sync Status:
```bash
# Detailed application status
argocd app get my-app --refresh

# Watch application sync
argocd app wait my-app --sync

# Show application resources
argocd app resources my-app

# Show application tree / operation state
argocd app get my-app --show-operation

# Get application events
kubectl get events -n argocd | grep my-app

# Describe application
kubectl describe app my-app -n argocd
```

## 🔧 Repository Management

### Repository Configuration

Add Repository:
```bash
# Add Git repository
argocd repo add https://github.com/user/repo.git \
  --username <username> \
  --password <password>

# Add private Git repository with SSH
argocd repo add git@github.com:user/repo.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# Add Helm repository
argocd repo add https://charts.example.com \
  --type helm \
  --name my-helm-repo

# Add OCI repository
argocd repo add oci://ghcr.io/user/charts \
  --type helm

# List repositories
argocd repo list

# Repository details
argocd repo get https://github.com/user/repo.git

# Remove repository
argocd repo rm https://github.com/user/repo.git
```

SSH Key Management:
```bash
# Generate SSH key for deployment
ssh-keygen -t rsa -b 4096 -f argocd-key -N ""

# Add SSH key to known_hosts
ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

# Create secret with SSH key
kubectl create secret generic argocd-ssh-key \
  -n argocd \
  --from-file=ssh-private-key=argocd-key

# Create secret with personal access token
kubectl create secret generic github-credentials \
  -n argocd \
  --from-literal=username=<username> \
  --from-literal=password=<token>
```

## 🔄 Synchronization Policies

### Sync Options & Automation

Auto Sync Configuration:
```bash
# Enable auto-sync
argocd app set my-app \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Disable auto-sync
argocd app set my-app \
  --sync-policy none

# Set sync window (only sync during maintenance window)
argocd app set my-app \
  --sync-window-allow "Mon-Sun 01:00-02:00"

# Deny sync during certain times
argocd app set my-app \
  --sync-window-deny "Mon-Fri 10:00-12:00"
```

Manual Sync:
```bash
# Sync with pruning
argocd app sync my-app --prune

# Sync with dry-run
argocd app sync my-app --dry-run

# Sync specific resource
argocd app sync my-app --resource deployment/my-deployment

# Sync and wait
argocd app sync my-app --wait

# Sync with retry
argocd app sync my-app --retry-limit 5

# Force sync (ignore differences)
argocd app sync my-app --force
```

Sync Policies YAML:
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
    allow:
      empty: false
  syncOptions:
  - CreateNamespace=true
  - PrunePropagationPolicy=foreground
  - RespectIgnoreDifferences=true
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
  passOperationOnFailure: false
```

## 🎯 Projects & RBAC

### Project Management

Create Project:
```bash
# Create default project (already exists)
argocd proj create default

# Create custom project
argocd proj create my-project \
  --description "My project" \
  --dest kubernetes.default.svc,default \
  --src https://github.com/user/repo.git

# Allow multiple destinations
argocd proj add-destination my-project \
  kubernetes.default.svc \
  production

# Allow multiple sources
argocd proj add-source my-project \
  https://github.com/user/repo.git

# List projects
argocd proj list

# Get project details
argocd proj get my-project

# Delete project
argocd proj delete my-project
```

Project Permissions:
```bash
# Allow cluster resource
argocd proj allow-cluster-resource my-project \
  "*" "*" "*"

# Deny cluster resource
argocd proj deny-cluster-resource my-project \
  "apps" "deployments" "*"

# Allow namespaced resource
argocd proj allow-namespace-resource my-project \
  "apps" "deployments" "true"

# Set project defaults
argocd proj set my-project \
  --orphaned-resources-warn true \
  --orphaned-resources-delete true
```

## 👥 User & Access Management

### User Configuration

User Management:
```bash
# List users
argocd account list

# Create local user
argocd account create alice

# Get user details
argocd account get alice

# Set user password
argocd account update-password \
  --account alice \
  --new-password <new-password>

# Disable user
argocd account disable alice

# Enable user
argocd account enable alice

# Create API token
argocd account generate-token --account alice

# Get API tokens
argocd account get alice
```

RBAC Configuration:
```bash
# Edit RBAC ConfigMap
kubectl edit configmap argocd-rbac-cm -n argocd
```
```ini
# Example RBAC policy
p, role:admin, *, *, */*, allow
p, role:readonly, applications, get, */*, allow
p, role:readonly, repositories, get, */*, allow
p, role:developers, applications, sync, my-project/*, allow
g, alice, role:developers
g, bob@example.com, role:admin
```

## 🔐 Secrets & Configuration

### Secrets Management

Repository Credentials:
```bash
# Add Git credentials via secret
kubectl create secret generic git-credentials \
  -n argocd \
  --from-literal=username=<username> \
  --from-literal=password=<password>

# Add SSH key
kubectl create secret generic git-ssh-key \
  -n argocd \
  --from-file=ssh-private-key=~/.ssh/id_rsa

# Add TLS certificate
kubectl create secret generic git-tls-cert \
  -n argocd \
  --from-file=tls.crt=/path/to/cert.pem

# Create registry credentials
kubectl create secret docker-registry \
  docker-registry-creds \
  -n argocd \
  --docker-server=docker.io \
  --docker-username=<username> \
  --docker-password=<password>
```

ArgoCD Configuration:
```bash
# Edit ArgoCD ConfigMap
kubectl edit configmap argocd-cm -n argocd
```
```yaml
# Common settings:
data:
  application.instanceLabelKey: argocd.argoproj.io/instance
  server.insecure: "false"
  accounts.alice: apiKey
  accounts.bob: apiKey,login
  url: https://argocd.example.com
  timeout.reconciliation: 180s
```
```bash
# Edit RBAC ConfigMap
kubectl edit configmap argocd-rbac-cm -n argocd

# Edit Notifications ConfigMap
kubectl edit configmap argocd-notifications-cm -n argocd
```

## 🔌 Notifications & Webhooks

### Notifications Setup

Slack Notifications:
```bash
# Edit notifications ConfigMap
kubectl edit configmap argocd-notifications-cm -n argocd
```
```yaml
# Add Slack configuration
service.slack: |
  token: $slack-token
```
```bash
# Create Slack token secret
kubectl create secret generic slack-token \
  -n argocd \
  --from-literal=slack-token=xoxb-...

# Subscribe to notifications
argocd notification subscribe slack:#deployments \
  my-app:sync:succeeded
```

Email Notifications:
```bash
# Configure email service
kubectl edit configmap argocd-notifications-cm -n argocd
```
```ini
service.email.host: smtp.gmail.com
service.email.port: "465"
service.email.from: noreply@example.com
service.email.username: $email-username
service.email.password: $email-password
```
```bash
# Create email credentials
kubectl create secret generic email-credentials \
  -n argocd \
  --from-literal=email-username=user@gmail.com \
  --from-literal=email-password=<password>
```

Webhook & Triggers:
```bash
# GitHub webhook trigger (auto-sync on push)
# Settings > Webhooks > Add webhook
# Payload URL: https://argocd.example.com/api/webhook

# Refresh git periodically
argocd app set my-app \
  --repo-refresh-timeout 300s

# Trigger sync webhook
curl -X POST https://argocd.example.com/api/webhook \
  -H "Content-Type: application/json" \
  -d '{"push":{}}'
```

## 📊 Monitoring & Logging

### Monitoring ArgoCD

Logging & Debugging:
```bash
# View ArgoCD server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server

# View application controller logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# View repo server logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server

# Follow logs
kubectl logs -n argocd -f -l app.kubernetes.io/name=argocd-server

# Debug application sync
argocd app get my-app --show-operation

# Enable verbose logging
kubectl set env deployment/argocd-server -n argocd LOG_LEVEL=debug
```

Metrics & Prometheus:
```bash
# ArgoCD exposes Prometheus metrics:
# Server metrics: :8083/metrics
# Repo server metrics: :8084/metrics
# Controller metrics: :8085/metrics

# Get metrics from server
kubectl port-forward svc/argocd-metrics -n argocd 8082:8082
```
```ini
# Sample Prometheus queries
argocd_app_sync_total            # total syncs
argocd_app_sync_duration_seconds # sync duration
argocd_git_request_total         # git requests
```

## 🛡️ Security Best Practices

### Security Configuration

SSL/TLS Configuration:
```bash
# Create self-signed certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Create secret from certificate
kubectl create secret tls argocd-server-tls \
  -n argocd \
  --cert=cert.pem \
  --key=key.pem

# Enable insecure mode (for testing only)
kubectl patch configmap argocd-cmd-params-cm -n argocd -p '{"data":{"server.insecure":"true"}}'
```

RBAC & Permissions:
```bash
# Create read-only role
argocd proj create readonly \
  --dest kubernetes.default.svc,default

# Create developer role
argocd proj create developers \
  --dest kubernetes.default.svc,development

# Restrict project resources
argocd proj add-destination my-project \
  kubernetes.default.svc \
  production \
  --allow-wildcard

# Create service account
kubectl create serviceaccount argocd-github-webhook -n argocd

# Create API token for service account
argocd account generate-token \
  --account argocd-github-webhook
```

Security Scanning:
```bash
# Enable image scanning with Trivy
argocd app set my-app \
  --security-scanning trivy

# Set pod security policy label
kubectl label pods argocd-server \
  -n argocd \
  pod-security.kubernetes.io/enforce=restricted
```

Network Policy:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: argocd-network-policy
  namespace: argocd
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## 🚀 Advanced Features

### Multi-Cluster & Advanced

Multi-Cluster Setup:
```bash
# Register additional cluster
argocd cluster add <context-name>

# List clusters
argocd cluster list

# Add cluster with credentials
argocd cluster add my-cluster \
  --name my-cluster \
  --in-cluster=false

# Remove cluster
argocd cluster rm my-cluster

# Deploy to multiple clusters
argocd app create multi-cluster-app \
  --repo https://github.com/user/repo.git \
  --path ./kubernetes \
  --dest-server https://cluster1.example.com:6443 \
  --dest-namespace default
```

GitOps Patterns:

App of Apps pattern:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    path: apps
    plugin:
      name: kustomize
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
```

Helm values override:
```bash
argocd app create helm-app \
  --repo https://github.com/user/repo.git \
  --path ./helm \
  --values values.yaml \
  --helm-set image.tag=v1.0.0
```

Kustomize patches:
```bash
argocd app create kustomize-app \
  --repo https://github.com/user/repo.git \
  --path ./kustomize
```

## 📋 Quick Reference Commands

| Command | Purpose | Example |
|---|---|---|
| `app create` | Create app | `argocd app create my-app --repo ... --path ...` |
| `app list` | List apps | `argocd app list` |
| `app get` | Get app details | `argocd app get my-app` |
| `app sync` | Sync app | `argocd app sync my-app` |
| `app delete` | Delete app | `argocd app delete my-app` |
| `repo add` | Add repository | `argocd repo add https://...` |
| `repo list` | List repos | `argocd repo list` |
| `proj create` | Create project | `argocd proj create my-proj` |
| `account list` | List users | `argocd account list` |
| `login` | Login to server | `argocd login server.com` |
| `version` | Check version | `argocd version` |
| `diff` | Compare diffs | `argocd app diff my-app` |

## ✅ Best Practices

**Repository Structure**
- Organize manifests by environment
- Use separate branches for environments
- Version control everything
- Use Kustomize/Helm for variations
- Keep secrets in sealed-secrets

**Application Management**
- Use projects for organization
- Enable auto-sync carefully
- Set appropriate sync windows
- Monitor sync status
- Use notifications

**Security**
- Use private repositories
- Implement RBAC properly
- Rotate API tokens regularly
- Enable TLS/SSL
- Audit access logs

**Operations**
- Set retry policies
- Configure health assessments
- Monitor cluster resources
- Perform regular backups
- Test disaster recovery

💡 **Pro Tips:**
- Use the app-of-apps pattern
- Leverage Kustomize overlays
- Implement sync windows
- Use pre-sync/post-sync hooks
- Enable self-healing

⚠️ **Avoid:**
- Manual `kubectl apply` against GitOps-managed resources
- Storing secrets in git
- Disabling RBAC
- Using insecure connections
- Ignoring sync failures

## 🎓 Common Patterns

**Deployment Strategies**
- Blue-Green
- Canary
- Rolling Updates
- Sync Waves
- Progressive Delivery

**Repository Patterns**
- Mono Repo
- Multi Repo
- App of Apps
- Helmfile
- Kustomize

**Sync Policies**
- Manual
- Auto Sync
- Self-Heal
- Sync Windows
- Retry Policy

---

*Source: adapted from the ArgoCD cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
