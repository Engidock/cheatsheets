# Kubernetes Cheatsheet

Complete quick reference guide for kubectl commands & configurations.

## 🔧 Cluster Management

### Cluster Info & Access

Get cluster info:
```bash
kubectl cluster-info
```

View current context:
```bash
kubectl config current-context
```

List all contexts:
```bash
kubectl config get-contexts
```

Switch context:
```bash
kubectl config use-context context-name
```

Check cluster resources:
```bash
kubectl top nodes
kubectl top pods
```

### Node Management

List all nodes:
```bash
kubectl get nodes -o wide
```

Describe node:
```bash
kubectl describe node node-name
```

Drain node:
```bash
kubectl drain node-name --ignore-daemonsets --delete-emptydir-data
```

Cordon/Uncordon node:
```bash
kubectl cordon node-name
kubectl uncordon node-name
```

Label node:
```bash
kubectl label nodes node-name key=value
```

## 📦 Pod Operations

### Pod Viewing & Inspection

List pods:
```bash
kubectl get pods
kubectl get pods -n namespace
kubectl get pods -A
```

Describe pod:
```bash
kubectl describe pod pod-name
```

Get pod YAML:
```bash
kubectl get pod pod-name -o yaml
```

### Pod Lifecycle

View pod logs:
```bash
kubectl logs pod-name
kubectl logs pod-name -f
kubectl logs pod-name --tail=100
```

Execute command:
```bash
kubectl exec -it pod-name -- /bin/bash
```

Delete pod:
```bash
kubectl delete pod pod-name
```

## 🚀 Deployments

### Deployment Management

List deployments:
```bash
kubectl get deployments
kubectl get deployments -A
```

Create deployment:
```bash
kubectl create deployment name --image=image:tag
kubectl apply -f deployment.yaml
```

Scale deployment:
```bash
kubectl scale deployment name --replicas=3
```

Update image:
```bash
kubectl set image deployment/name container=image:tag
```

Rollout operations:
```bash
kubectl rollout status deployment/name
kubectl rollout undo deployment/name
```

## 🌐 Services & Networking

### Service Operations

List services:
```bash
kubectl get services
kubectl get svc -A
```

Create service:
```bash
kubectl expose deployment name --type=LoadBalancer --port=8080
```

Port forward:
```bash
kubectl port-forward svc/service-name 8080:8080
```

List ingresses:
```bash
kubectl get ingress -A
```

## 🔐 Configuration & Secrets

### ConfigMap & Secrets

List config maps:
```bash
kubectl get configmaps
```

Create config map:
```bash
kubectl create configmap name --from-literal=key=value
```

List secrets:
```bash
kubectl get secrets
```

Create secret:
```bash
kubectl create secret generic name --from-literal=key=value
```

Decode secret:
```bash
kubectl get secret name -o jsonpath='{.data.key}' | base64 -d
```

## 💾 Storage

### Persistent Volumes

List PV and PVC:
```bash
kubectl get pv
kubectl get pvc
```

Storage classes:
```bash
kubectl get storageclass
```

## 👥 Namespace & RBAC

### Namespace & Access Control

List namespaces:
```bash
kubectl get namespaces
```

Create namespace:
```bash
kubectl create namespace name
```

Set default namespace:
```bash
kubectl config set-context --current --namespace=prod
```

List roles:
```bash
kubectl get roles
kubectl get clusterroles
```

Check permissions:
```bash
kubectl auth can-i get pods
```

## ⚙️ Resource Management

### Resources & Scaling

View resource usage:
```bash
kubectl top nodes
kubectl top pods
```

Resource quotas:
```bash
kubectl get resourcequota
```

List HPA:
```bash
kubectl get hpa
```

Create HPA:
```bash
kubectl autoscale deployment name --min=2 --max=10 --cpu-percent=80
```

## 🔍 Debugging & Monitoring

### Logs & Troubleshooting

View events:
```bash
kubectl get events -n namespace
```

Debug pod:
```bash
kubectl exec -it pod-name -- /bin/bash
```

Port forwarding:
```bash
kubectl port-forward svc/service 8080:8080
```

Test DNS:
```bash
kubectl run -it --rm debug --image=busybox -- nslookup service
```

## ⚡ Advanced Commands

### Filtering & Output

Output formats:
```bash
kubectl get pods -o json
kubectl get pods -o yaml
kubectl get pods -o wide
```

Label selectors:
```bash
kubectl get pods -l app=myapp
kubectl get pods -l app=myapp,tier=frontend
```

JSONPath extraction:
```bash
kubectl get pod mypod -o jsonpath='{.status.phase}'
```

Bulk operations:
```bash
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl edit pod pod-name
kubectl get pods -w
```

### 📋 Quick Reference Flags

| Flag | Example | Description |
|------|---------|-------------|
| `-n` | `kubectl get pods -n prod` | Specify namespace |
| `-A` | `kubectl get pods -A` | All namespaces |
| `-l` | `kubectl get pods -l app=myapp` | Filter by labels |
| `-o` | `kubectl get pods -o yaml` | Output format |
| `-w` | `kubectl get pods -w` | Watch changes |
| `--dry-run` | `kubectl apply -f file --dry-run=client` | Preview changes |
| `--record` | `kubectl set image deploy/name img --record` | Record command |
| `--cascade` | `kubectl delete deploy --cascade=orphan` | Delete with children |

### 📚 Kubernetes Resource Types

**Workloads**
- Pod
- Deployment
- StatefulSet
- DaemonSet
- Job
- CronJob

**Services & Networking**
- Service
- Ingress
- NetworkPolicy

**Configuration**
- ConfigMap
- Secret
- ServiceAccount

**Storage**
- PersistentVolume
- PersistentVolumeClaim
- StorageClass

## ✅ Best Practices

- **Always use namespaces** - Organize resources logically and isolate workloads
- **Use labels effectively** - Label all resources for organization and selection
- **Set resource requests & limits** - Ensure proper scheduling and resource allocation
- **Configure health checks** - Use liveness and readiness probes for reliability
- **Version control manifests** - Store all YAML in Git for GitOps workflow
- **Use dry-run before apply** - Preview changes:
```bash
kubectl apply -f file.yaml --dry-run=client
```

### Troubleshooting Checklist

**🔍 Pod not starting?**
1. Check pod events:
   ```bash
   kubectl describe pod pod-name
   ```
2. Check logs:
   ```bash
   kubectl logs pod-name
   ```
3. Check previous logs:
   ```bash
   kubectl logs pod-name --previous
   ```
4. Check node resources:
   ```bash
   kubectl top nodes
   ```

**📊 High resource usage?**
1. View usage:
   ```bash
   kubectl top nodes && kubectl top pods
   ```
2. Check limits:
   ```bash
   kubectl get pod pod-name -o yaml | grep -A 10 resources
   ```
3. Check HPA:
   ```bash
   kubectl describe hpa hpa-name
   ```

**🌐 Connectivity issues?**
1. Test DNS:
   ```bash
   kubectl run -it --rm debug --image=busybox -- nslookup service
   ```
2. Check endpoints:
   ```bash
   kubectl get endpoints service-name
   ```
3. Check network policies:
   ```bash
   kubectl get netpol
   ```

### 💡 Useful Aliases

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
```

---

*Source: adapted from the Kubernetes cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
