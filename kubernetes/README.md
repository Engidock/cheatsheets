# Kubernetes Cheatsheet

> Replace this placeholder with your actual EngiDock Kubernetes cheatsheet content (or copy it from engidock.com/cheatsheets).

## Common kubectl commands

```bash
# Get all pods in the current namespace
kubectl get pods

# Describe a pod (events, status, errors)
kubectl describe pod <pod-name>

# Tail logs from a pod
kubectl logs -f <pod-name>

# Exec into a running container
kubectl exec -it <pod-name> -- /bin/sh

# Apply a manifest
kubectl apply -f deployment.yaml

# Roll back a deployment
kubectl rollout undo deployment/<name>
```

## Troubleshooting checklist

| Symptom | Check |
|---|---|
| Pod stuck in `Pending` | `kubectl describe pod` for scheduling/resource errors |
| Pod in `CrashLoopBackOff` | `kubectl logs --previous` for the last crash reason |
| Service unreachable | Confirm selector labels match pod labels |

---
*Full course: [engidock.com/course/7](https://www.engidock.com/course/7)*
