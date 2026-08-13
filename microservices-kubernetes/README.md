# Microservices & Kubernetes Cheatsheet

Comprehensive Cheatsheet & Quick Reference Guide
Complete reference for building, deploying, and managing microservices on Kubernetes.

## 🧩 1. Core Concepts

### Microservices Definition

Microservices: an architectural approach where an application is built as a collection of small, independent services communicating through APIs.

**Key Characteristics:**
- Single responsibility - each service owns one capability
- Independent deployment - services deploy separately
- Database per service - avoid shared databases
- Loosely coupled - minimal dependencies
- Scalable independently - scale services individually
- Fault isolated - failures don't cascade

### Monolith vs Microservices

| Aspect | Monolith | Microservices |
|---|---|---|
| Codebase | Single large codebase | Multiple separate codebases |
| Database | Shared database | Database per service |
| Deployment | Deploy entire application | Deploy individual services |
| Scaling | Scale entire application | Scale individual services |
| Technology | Single tech stack | Polyglot (mixed stacks) |
| Complexity | Lower initial complexity | Higher distributed complexity |

### When to Use Microservices

**Good for:**
- Large applications with multiple teams
- Independently scalable components
- Multiple technology requirements
- Frequent independent deployments
- High availability requirements

**Avoid if:**
- Small team (<5 people)
- Simple monolithic requirements
- No independent scaling needs
- Low deployment frequency
- Strong consistency requirements

## 🔗 2. Microservices Patterns

### Communication Patterns

**REST API Pattern**

| Characteristic | Details |
|---|---|
| Type | Synchronous, request-response |
| Coupling | Tight (caller waits for response) |
| Reliability | Low (service down = no response) |
| Latency | Low (immediate response) |
| Best for | Simple queries, immediate feedback |
| Implementation | HTTP/HTTPS with JSON/XML |

**Message Queue Pattern**

```python
message = {
    'order_id': '123',
    'customer_id': 'cust-456',
    'total': 99.99
}
queue.send('order-events', message)

for message in queue.consume('order-events'):
    process_order(message)
    queue.acknowledge(message)
```

**Event-Driven Pattern**

```python
event = {'event': 'OrderCreated', 'order_id': '123'}
event_bus.publish('order-events', event)

# Subscriber 1: Payment Service
@event_bus.subscribe('order-events')
def process_payment(event):
    if event['event'] == 'OrderCreated':
        charge_payment(event['order_id'])

# Subscriber 2: Notification Service
@event_bus.subscribe('order-events')
def send_notification(event):
    if event['event'] == 'OrderCreated':
        send_email(event['order_id'])
```

### Resilience Patterns

**Circuit Breaker**

- **CLOSED** - Calls pass through normally. Transitions to OPEN when the failure threshold is exceeded.
- **OPEN** - Calls fail immediately (fast fail). Transitions to HALF_OPEN after a timeout.
- **HALF_OPEN** - Allows a limited number of test calls through. Transitions to CLOSED on success, back to OPEN on failure.

**Retry Pattern with Backoff**

```python
def call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except TransientError as e:
            if attempt < max_retries - 1:
                wait_time = (2 ** attempt) + random(0, 1)
                time.sleep(wait_time)
            else:
                raise
```

**Timeout Pattern**

```python
# Service A calling Service B with timeout
timeout_a = 5000  # ms
timeout_b = 3000  # child service should be faster
timeout_c = 2000  # grandchild service

response = requests.get(
    'http://service-b/api/data',
    timeout=timeout_b / 1000
)
```

### Data Patterns

**Saga Pattern (Distributed Transactions)**

```python
class OrderSaga:
    def execute_order(order):
        # Step 1: Reserve inventory
        reservation = inventory.reserve(order.items)
        if not reservation:
            compensate()
            return

        # Step 2: Process payment
        payment = payment_service.charge(order.total)
        if not payment:
            inventory.release(reservation)
            return

        # Step 3: Confirm order
        order.confirm()

        # If any step fails, compensation logic runs
```

**Event Sourcing Pattern**
- Store every state change as an immutable event
- Replay events to reconstruct current state
- Complete audit trail of all changes
- Can view state at any point in time

**CQRS Pattern (Command Query Responsibility Segregation)**
- Separate read and write models
- Write model optimized for consistency
- Read model optimized for queries
- Eventual consistency between models
- Independent scaling of reads and writes

## 🐳 3. Docker Essentials

### Docker Workflow

**Build Image**

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt
COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

```bash
docker build -t myapp:1.0 .

docker build -t myapp:1.0 --no-cache .
```

**Run Container**

```bash
docker run -d --name myapp-container myapp:1.0

docker run -d -p 8080:8080 --name myapp myapp:1.0
docker run -d -p 8080:8080 -e DB_HOST=db.local myapp:1.0
docker run -it myapp:1.0 /bin/bash
```

### Docker Commands Reference

Build image from Dockerfile
```bash
docker build -t tag .
```

Run container in background
```bash
docker run -d image
```

List running containers
```bash
docker ps
```

List all containers
```bash
docker ps -a
```

View container logs
```bash
docker logs container
```

Execute command in container
```bash
docker exec -it container bash
```

Stop running container
```bash
docker stop container
```

Push image to registry
```bash
docker push registry/image:tag
```

Remove container
```bash
docker rm container
```

Remove image
```bash
docker rmi image
```

### Docker Best Practices
- Use specific base image tags (avoid `latest`)
- Minimize image size (use slim/alpine versions)
- Don't run as root (use `USER` instruction)
- Layer caching - put frequently changing items last
- Use `.dockerignore` to exclude unnecessary files
- Health check - add `HEALTHCHECK` instruction
- Multi-stage builds - separate build and runtime stages
- Environment variables - avoid hardcoding config

## ☸️ 4. Kubernetes Fundamentals

### Kubernetes Architecture

| Component | Description |
|---|---|
| API Server | Central control point; all communication goes through it |
| Scheduler | Assigns pods to nodes based on resource requirements |
| etcd | Distributed key-value store for cluster state |
| Controller Manager | Runs controllers (Deployment, Service, etc.) |
| Kubelet | Node agent ensuring containers run in pods |
| kube-proxy | Network proxy, maintains network rules |

### Core Resources

**Pod (Smallest Unit)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
```

**Deployment (Stateless Workload)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: myapp:1.0
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
```

**Service (Network Access)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

**ConfigMap & Secret**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "postgres.default.svc.cluster.local"
  DATABASE_PORT: "5432"
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DATABASE_PASSWORD: cGFzc3dvcmQ=  # base64 encoded
```

**Ingress (HTTP Routing)**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp
                port:
                  number: 80
    - host: api.example.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: myapp-api
                port:
                  number: 8080
```

## ⌨️ 5. Kubectl Commands Reference

### Basic Commands

Display cluster information
```bash
kubectl cluster-info
```

List all nodes
```bash
kubectl get nodes
```

List all pods in all namespaces
```bash
kubectl get pods -A
```

List all deployments
```bash
kubectl get deployments
```

List all services
```bash
kubectl get svc
```

List all ConfigMaps
```bash
kubectl get configmap
```

List persistent volume claims
```bash
kubectl get pvc
```

### Create & Apply

Apply/update resource from manifest
```bash
kubectl apply -f manifest.yaml
```

Create deployment imperatively
```bash
kubectl create deployment app --image=app:1.0
```

Expose deployment as service
```bash
kubectl expose deployment app --port=80
```

Update image in deployment
```bash
kubectl set image deployment/app app=app:2.0
```

### Describe & Debug

Show detailed pod information
```bash
kubectl describe pod pod-name
```

View pod logs
```bash
kubectl logs pod-name
```

Follow pod logs (tail)
```bash
kubectl logs -f pod-name
```

View specific container logs
```bash
kubectl logs pod-name -c container
```

Execute command in pod
```bash
kubectl exec -it pod-name bash
```

Forward local port to pod
```bash
kubectl port-forward pod-name 8080:8080
```

Show node resource usage
```bash
kubectl top nodes
```

Show pod resource usage
```bash
kubectl top pods
```

### Rollout & Updates

View deployment history
```bash
kubectl rollout history deployment/app
```

Check rollout status
```bash
kubectl rollout status deployment/app
```

Rollback to previous version
```bash
kubectl rollout undo deployment/app
```

Scale deployment to 5 replicas
```bash
kubectl scale deployment/app --replicas=5
```

### Delete Resources

Delete specific pod
```bash
kubectl delete pod pod-name
```

Delete deployment (cascades to pods)
```bash
kubectl delete deployment app
```

Delete resources from manifest
```bash
kubectl delete -f manifest.yaml
```

## 📐 6. Service Design Guidelines

### Service Sizing

**Too Small Services - Problems:**
- Increased operational overhead
- Complex inter-service communication
- Distributed transactions everywhere
- Network latency becomes a bottleneck

**Appropriate Service Size - Guidelines:**
- One team (2-8 engineers) per service
- Handles one business capability
- Loosely coupled with other services
- Can be rebuilt in 1-2 weeks
- Clear API contract with other services

### API Design

**RESTful API Best Practices**

| Aspect | Avoid | Prefer |
|---|---|---|
| Resources | `GET /getUser?id=123` | `GET /users/123` |
| Methods | `POST /createUser` | `POST /users` (create) |
| Status Codes | `200 OK` for all responses | `201 Created`, `404 Not Found`, etc. |
| Versioning | No version in API | `/v2/users` or `Accept: application/vnd.api+json;version=2` |
| Pagination | No pagination | `?page=1&limit=20` |

### Database Strategy

**Database Per Service - Benefits:**
- Service independence - can use appropriate database type
- Avoid tight coupling through shared data
- Scale databases independently
- Choose best database for use case

**Data Sharing Between Services**

| Pattern | Best Use Case | Complexity | Consistency |
|---|---|---|---|
| REST API | Simple queries | Simple | Strong (sync) |
| Message Queue | Asynchronous updates | Medium | Eventual |
| Event Sourcing | Complete audit trail | Complex | Eventual |
| Caching | Frequently accessed data | Medium | Eventual |

## 🚀 7. Deployment Strategies

### Rolling Update

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Additional pods during update
      maxUnavailable: 0  # Zero downtime
```

### Blue-Green Deployment

1. **Setup** - Deploy new version (green) alongside old (blue)
2. **Test** - Run tests against green environment
3. **Switch** - Instantly switch load balancer from blue to green
4. **Rollback** - Switch back to blue if issues detected

### Canary Deployment

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
    - myapp
  http:
    - match:
        - uri:
            prefix: /
      route:
        - destination:
            host: myapp
            subset: v1
          weight: 90
        - destination:
            host: myapp
            subset: v2
          weight: 10
```

### Deployment Comparison

| Strategy | Downtime | Rollback Time | Risk | Best For |
|---|---|---|---|---|
| Rolling | Zero | Slow | Medium | Standard deployments |
| Blue-Green | Zero | Instant | Low | Critical services |
| Canary | Zero | Fast | Very Low | Risky changes |

## 📊 8. Monitoring & Debugging

### Key Metrics to Monitor

| Metric | What to Measure | Alert Threshold |
|---|---|---|
| Response Time (Latency) | P50, P95, P99 | P95 > 500ms, P99 > 1000ms |
| Error Rate | 4xx, 5xx responses | > 1% of requests |
| CPU Usage | CPU percentage per pod | > 80% sustained |
| Memory Usage | Memory percentage per pod | > 85% |
| Request Rate | Requests per second | Unusual spikes |
| Disk Space | Disk usage on nodes | > 80% |

### Debugging Checklist

**Pod Not Starting**

```bash
kubectl describe pod pod-name
kubectl logs pod-name
kubectl logs pod-name --previous   # Previous crashed container
kubectl get events --sort-by='.lastTimestamp'
```

**High Latency**

```bash
kubectl top pods
kubectl top nodes
kubectl logs pod-name | grep "duration"
```

```python
import time
start = time.time()
# ... operation ...
duration = time.time() - start
if duration > 1.0:
    logging.warning(f'Slow operation: {duration}s')
```

**Service Communication Issues**

```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  nslookup service-name

kubectl run -it --rm debug --image=busybox --restart=Never -- \
  wget -O- http://service-name:8080/health

kubectl get endpoints service-name
```

### Logging Best Practices
- Use structured logging (JSON format)
- Include correlation IDs for tracing requests
- Log at appropriate levels (DEBUG, INFO, WARN, ERROR)
- Aggregate logs centrally (ELK, Splunk, etc.)
- Set appropriate log retention policies
- Don't log sensitive data (passwords, tokens)

### Monitoring Stack Components

| Component | Purpose | Examples |
|---|---|---|
| Metrics Collection | Collect time-series metrics | Prometheus, Datadog, New Relic |
| Metrics Storage | Store and query metrics | Prometheus, InfluxDB, CloudWatch |
| Visualization | Display metrics in dashboards | Grafana, Kibana, CloudWatch |
| Log Aggregation | Collect logs from all services | ELK Stack, Splunk, Datadog |
| Tracing | Track requests across services | Jaeger, Zipkin, Datadog APM |
| Alerting | Alert on metric thresholds | Prometheus AlertManager, PagerDuty |

## 🔒 9. Security Checklist

### Pod Security

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true            # Don't run as root
    runAsUser: 1000               # Run as specific user
    readOnlyRootFilesystem: true  # Read-only filesystem
  containers:
    - name: app
      image: app:1.0
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
      resources:
        limits:
          cpu: "500m"
          memory: "256Mi"
```

### Network Security

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: api
      ports:
        - protocol: TCP
          port: 8080
```

### RBAC (Role-Based Access Control)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: app-sa
```

### Secrets Management

**Best Practices:**
- Never store secrets in ConfigMaps
- Use encrypted secrets storage (etcd encryption)
- Use external secrets management (HashiCorp Vault, AWS Secrets Manager)
- Rotate secrets regularly
- Limit access to secrets with RBAC
- Audit access to secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: app
      image: app:1.0
      volumeMounts:
        - name: secret-vol
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-vol
      secret:
        secretName: app-secret
```

### Security Checklist
- [x] Run containers as non-root user
- [x] Use read-only root filesystem
- [x] Drop unnecessary Linux capabilities
- [x] Set resource limits (CPU, memory)
- [x] Use network policies to restrict traffic
- [x] Implement RBAC with minimal permissions
- [x] Encrypt secrets in etcd
- [x] Use Pod Security Policies / Pod Security Standards
- [x] Enable audit logging
- [x] Scan images for vulnerabilities
- [x] Use private container registries
- [x] Enable authentication/authorization on API

## ⚡ 10. Performance Optimization

### Resource Management

```yaml
resources:
  requests:   # Guaranteed minimum
    cpu: 250m
    memory: 256Mi
  limits:     # Maximum allowed
    cpu: 500m
    memory: 512Mi
```

### Scaling Strategies

| Strategy | What It Does | Best For |
|---|---|---|
| Horizontal Pod Autoscaler (HPA) | Scale replicas based on metrics | Load spikes, variable traffic |
| Vertical Pod Autoscaler (VPA) | Adjust resource requests/limits based on usage | Optimize resource usage |
| Cluster Autoscaling | Add nodes when pods can't fit | Node capacity issues |

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Caching Strategies

| Strategy | How It Works | Typical TTL |
|---|---|---|
| Cache-Aside | Check cache first, miss = fetch from DB | 5-10 minutes |
| Write-Through | Update cache when writing to DB | Indefinite until invalidated |
| Write-Behind | Write to cache immediately, async to DB | Indefinite until flushed |

### Database Optimization
- **Indexing**: Index frequently queried columns
- **Query Optimization**: Use EXPLAIN ANALYZE
- **Connection Pooling**: Reuse database connections
- **Read Replicas**: Offload reads from primary
- **Sharding**: Distribute data across multiple databases
- **Pagination**: Limit results, don't load all data

### Network Optimization
- **Batch Requests**: Combine multiple operations
- **Connection Reuse**: Keep connections open
- **Compression**: Gzip responses
- **Async Operations**: Don't block on slow operations
- **Circuit Breaker**: Fail fast on unavailable services
- **Caching**: Cache frequently accessed resources

## 🛠️ 11. Troubleshooting Common Issues

### Pod Issues

| Status | Symptom | Solution |
|---|---|---|
| ImagePullBackOff | Pod stuck in pending | Check image exists, registry access, imagePullSecrets |
| CrashLoopBackOff | Pod constantly restarting | Check logs, fix entrypoint/config |
| Pending | Pod not scheduling | Check resources, node selectors, taints/tolerations |
| OOMKilled | Memory limit exceeded | Increase memory limit or optimize code |

### Service Communication

```bash
# Check if service exists
kubectl get svc service-name

# Check service endpoints (backing pods)
kubectl get endpoints service-name

# Test DNS from pod
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  nslookup service-name.namespace.svc.cluster.local

# Test HTTP connectivity
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl http://service-name:port/health
```

### Storage Issues

| Issue | Symptom | Solution |
|---|---|---|
| PersistentVolumeClaim Pending | No available PV | Create PV or use dynamic provisioning |
| Pod Can't Mount Volume | PVC in different namespace | Ensure PVC exists in pod namespace |
| Disk Full | PVC capacity exceeded | Increase PV size or clean up data |

### Networking Issues

| Problem | Diagnosis | Solution |
|---|---|---|
| Connection Refused | Service not listening on port | Check service port and container port match |
| Timeout | Network policies blocking or no route | Check network policies, routing |
| DNS Not Resolving | CoreDNS issue or service doesn't exist | Check CoreDNS pods, service name |

## 📋 12. Essential Quick Reference

### Microservices Checklist

**Design Phase**
- [ ] Define service boundaries clearly
- [ ] Plan data ownership and sharing
- [ ] Choose communication patterns
- [ ] Design APIs (versioning, contracts)
- [ ] Plan error handling and retries

**Implementation Phase**
- [ ] Implement health checks
- [ ] Add structured logging with correlation IDs
- [ ] Implement circuit breakers
- [ ] Add authentication/authorization
- [ ] Write tests (unit, integration, end-to-end)

**Deployment Phase**
- [ ] Containerize with Docker
- [ ] Create Kubernetes manifests
- [ ] Set resource limits
- [ ] Configure health checks
- [ ] Plan storage (if needed)

**Operations Phase**
- [ ] Set up monitoring and alerting
- [ ] Configure centralized logging
- [ ] Plan incident response
- [ ] Document runbooks
- [ ] Test disaster recovery

### Common Kubectl Aliases

```bash
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias krr='kubectl rollout restart'
```

### Key Formulas & Calculations

| Calculation | Formula | Example |
|---|---|---|
| Pod Capacity per Node | Node Memory / Pod Request | 32GB / 256Mi = ~128 pods |
| Replica Count for Scale | Peak RPS / RPS per Pod | 1000 RPS / 100 RPS = 10 replicas |
| HPA Max Replicas | Min Replicas + Headroom | 3 base + 5 extra = 8 max |
| Storage Growth | Data/Day x Days to Retain | 100GB/day x 30 days = 3TB |

## 📚 Additional Resources

- **Official Docs**: kubernetes.io/docs, docker.com/docs
- **Practice**: Play with Kubernetes on katacoda.com
- **Certification**: CKA, CKAD exams
- **Community**: CNCF, Kubernetes Slack, Stack Overflow
- **Tools**: Helm, Kustomize, Skaffold, Tilt

---

*Source: adapted from the Microservices & Kubernetes cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
