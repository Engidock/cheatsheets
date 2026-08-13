# Spinnaker Cheatsheet

Comprehensive, practical reference guide for Spinnaker — the open-source, multi-cloud continuous delivery platform.

## 🏗️ Spinnaker Architecture Quick Reference

### Overview

| Service | Port | Purpose | Key Responsibility |
|---|---|---|---|
| Deck | 9000 | Web UI | User interface for Spinnaker |
| Gate | 8084 | API Gateway | All API calls go through Gate |
| Orca | 8083 | Orchestration | Pipeline and task execution engine |
| Clouddriver | 7002 | Cloud Provider | Interfaces with AWS, GCP, Azure, K8s |
| Front50 | 8080 | Metadata Storage | Applications, pipelines, projects |
| Rosco | 8087 | Image Bakery | Bakes machine images (AMIs) |
| Igor | 8088 | CI Integration | Integrates with Jenkins, Travis, etc. |
| Echo | 8089 | Eventing | Notifications and event processing |
| Fiat | 7003 | Authorization | RBAC and permissions |
| Kayenta | 8090 | Canary Analysis | Automated canary analysis |
| Redis | 6379 | Cache | Distributed caching layer |

## 🛠️ Halyard Commands Reference

### Installation & Setup

```bash
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install.sh
sudo bash install.sh
hal --version

hal config version list
hal config version edit --version 1.35.0
sudo update-halyard
```

### Cloud Provider Configuration

```bash
hal config provider aws enable

hal config provider aws account add my-aws-account \
  --account-id 123456789012 \
  --assume-role role/SpinnakerRole \
  --regions us-east-1,us-west-2

hal config provider kubernetes enable

hal config provider kubernetes account add my-k8s-cluster \
  --context $(kubectl config current-context) \
  --docker-registries my-docker-registry

hal config provider google enable

hal config provider google account add my-gcp-account \
  --project my-project-id \
  --json-path /path/to/service-account.json

hal config provider azure enable

hal config provider azure account add my-azure-account \
  --client-id CLIENT_ID \
  --tenant-id TENANT_ID \
  --subscription-id SUBSCRIPTION_ID \
  --app-key APP_KEY \
  --default-resource-group my-rg \
  --default-key-vault my-vault

hal config provider aws account list

hal config provider kubernetes account list
```

### Storage Configuration

```bash
hal config storage s3 edit \
  --access-key-id YOUR_KEY \
  --secret-access-key \
  --region us-west-2 \
  --bucket spinnaker-artifacts

hal config storage edit --type s3

hal config storage gcs edit \
  --project my-project \
  --bucket-location us-central1 \
  --json-path /path/to/service-account.json

hal config storage edit --type gcs

hal config storage azs edit \
  --storage-account-name myaccount \
  --storage-account-key mykey \
  --storage-container-name spinnaker

hal config storage edit --type azs

hal config storage edit --type in-memory
```

### Deployment Configuration

```bash
hal config deploy edit \
  --type kubernetes \
  --kubernetes-namespace spinnaker \
  --account-name my-k8s-cluster

hal config deploy edit --type distributed

hal config deploy edit --type localdebian

hal deploy apply

hal deploy connect

hal deploy clean
```

### Security & RBAC

```bash
hal config security authz enable

hal config security authn ldap edit \
  --url ldap://ldap.company.com:389 \
  --user-dn-pattern "uid={0},ou=users,dc=company,dc=com"

hal config security authn oauth2 edit \
  --client-id CLIENT_ID \
  --client-secret CLIENT_SECRET \
  --provider google

hal config security ui edit \
  --override-base-url https://spinnaker.company.com

hal config security api edit \
  --override-base-url https://spinnaker-api.company.com

hal config security authz edit \
  --admin-group spinnaker-admins
```

### CI/CD Integration

```bash
hal config ci jenkins enable

hal config ci jenkins master add my-jenkins \
  --address https://jenkins.company.com \
  --username admin \
  --password

hal config provider docker-registry enable

hal config provider docker-registry account add my-docker-registry \
  --address https://index.docker.io \
  --repositories library/nginx,myorg/myapp \
  --username myuser \
  --password

hal config artifact github enable

hal config artifact github account add my-github \
  --token GITHUB_TOKEN
```

### Feature Configuration

```bash
hal config features edit --chaos true

hal config features edit --pipeline-templates true

hal config features edit --managed-pipeline-templates-v2-ui true

hal config features edit --artifacts-rewrite true

hal config features
```

### Monitoring & Logging

```bash
hal config metric-stores datadog enable

hal config metric-stores datadog edit \
  --api-key YOUR_API_KEY \
  --app-key YOUR_APP_KEY

hal config metric-stores prometheus enable

hal config metric-stores prometheus edit \
  --push-gateway localhost:9091

hal config metric-stores stackdriver enable

hal config metric-stores stackdriver edit \
  --credentials-path /path/to/credentials.json
```

### Service Configuration

```bash
hal config edit --timezone America/Los_Angeles

mkdir -p ~/.hal/default/profiles

echo "server.port: 8084" > ~/.hal/default/profiles/gate-local.yml

hal config deploy edit --deployment-environment k8s

hal config deploy ha clouddriver enable

hal config deploy ha echo enable

hal config deploy ha orca enable

hal config validate

hal backup create

hal backup restore --backup-path /path/to/backup.tar
```

## 📋 Pipeline Stages Quick Reference

### Bake Stage

Purpose: Create machine images (AMIs, Docker)
Type: `bake`
Use Cases: Image creation, artifact packaging

```json
{
  "type": "bake",
  "cloudProviderType": "aws",
  "baseOs": "ubuntu",
  "baseLabel": "release",
  "package": "myapp"
}
```

### Deploy Stage

Purpose: Deploy server groups to cloud providers
Type: `deploy`
Use Cases: Blue/Green, Canary, Rolling deployments

```json
{
  "type": "deploy",
  "clusters": [{
    "account": "my-aws-account",
    "capacity": {
      "desired": 3,
      "max": 6,
      "min": 3
    },
    "strategy": "redblack"
  }]
}
```

### Manual Judgment Stage

Purpose: Human approval before proceeding
Type: `manualJudgment`
Use Cases: Production approvals, QA gates

```json
{
  "type": "manualJudgment",
  "instructions": "Review before prod",
  "judgmentInputs": ["Approve", "Reject"],
  "failPipeline": true
}
```

### Webhook Stage

Purpose: Call external HTTP endpoints
Type: `webhook`
Use Cases: External integrations, custom logic

```json
{
  "type": "webhook",
  "url": "https://api.example.com",
  "method": "POST",
  "payload": {
    "app": "${application}"
  }
}
```

### Wait Stage

Purpose: Pause pipeline execution
Type: `wait`
Use Cases: Bake time, warm-up periods

```json
{
  "type": "wait",
  "waitTime": 300,
  "skipRemainingWait": false
}
```

### Check Preconditions Stage

Purpose: Conditional execution based on expressions
Type: `checkPreconditions`
Use Cases: Validation, conditional logic

```json
{
  "type": "checkPreconditions",
  "preconditions": [{
    "type": "expression",
    "context": {
      "expression": "${trigger.type == 'manual'}"
    }
  }]
}
```

### Find Image Stage

Purpose: Locate images from cloud provider
Type: `findImageFromTags`
Use Cases: Dynamic image selection

```json
{
  "type": "findImageFromTags",
  "cloudProvider": "aws",
  "tags": {
    "app": "myapp",
    "env": "prod"
  }
}
```

### Run Job Stage (Kubernetes)

Purpose: Execute Kubernetes jobs
Type: `runJob`
Use Cases: Database migrations, batch processing

```json
{
  "type": "runJob",
  "account": "my-k8s",
  "manifest": {
    "apiVersion": "batch/v1",
    "kind": "Job"
  }
}
```

### Scale Server Group Stage

Purpose: Change instance count
Type: `resizeServerGroup`
Use Cases: Scaling, capacity management

```json
{
  "type": "resizeServerGroup",
  "action": "scale_exact",
  "capacity": {
    "desired": 10
  }
}
```

### Destroy Server Group Stage

Purpose: Delete old server groups
Type: `destroyServerGroup`
Use Cases: Cleanup, cost optimization

```json
{
  "type": "destroyServerGroup",
  "target": "ancestor_asg_dynamic",
  "regions": ["us-east-1"]
}
```

### Enable/Disable Server Group Stage

Purpose: Control traffic to server groups
Type: `enableServerGroup` / `disableServerGroup`
Use Cases: Traffic management, rollbacks

```json
{
  "type": "enableServerGroup",
  "target": "current_asg_dynamic",
  "regions": ["us-east-1"]
}
```

### Kayenta Canary Stage

Purpose: Automated canary analysis
Type: `kayentaCanary`
Use Cases: Progressive delivery, safe deployments

```json
{
  "type": "kayentaCanary",
  "canaryConfig": {
    "scoreThresholds": {
      "marginal": 75,
      "pass": 90
    }
  }
}
```

## 🧮 SpEL (Spring Expression Language) Reference

### Common SpEL Expressions

Trigger context:

```text
${trigger.type}
${trigger.buildInfo.number}
${trigger.buildInfo.url}
${trigger.payload.repository.name}
${trigger.payload.pusher.name}
```

Execution context:

```text
${execution.id}
${execution.application}
${execution.name}
${execution.user}
${execution.startTime}
```

Pipeline parameters:

```text
${parameters.environment}
${parameters.version}
${parameters['param-with-dash']}
```

Stage reference:

```text
${#stage('Deploy to Dev')['status']}
${#stage('Bake')['context']['artifacts'][0]['reference']}
${#stage('Find Image')['context']['amiDetails'][0]['imageId']}
```

Conditional (ternary) logic:

```text
${parameters.env == 'production' ? 'prod-cluster' : 'dev-cluster'}
${trigger.buildInfo.number > 100 ? 'new' : 'old'}
${#stage('Previous Stage')['status'] == 'SUCCEEDED'}
```

String functions:

```text
${parameters.version.toUpperCase()}
${parameters.name.toLowerCase()}
${'v' + parameters.version}
${parameters.name.substring(0, 5)}
${parameters.url.contains('github')}
```

Math operations:

```text
${parameters.count + 5}
${parameters.instances * 2}
${parameters.timeout / 60}
```

Spinnaker helper functions:

```text
${#alphanumerical(parameters.name)}
${#toJson(parameters.config)}
${#fromUrl('https://api.example.com/status')}
${#toBoolean(parameters.enabled)}
${#toInt(parameters.count)}
${#toFloat(parameters.percentage)}
```

Date/time helpers:

```text
${new java.text.SimpleDateFormat('yyyy-MM-dd').format(new java.util.Date())}
${new java.util.Date().getTime()}
```

Artifacts, collections & null-safety:

```text
${artifacts[0].reference}
${#stage('Deploy')['context']['deployedServerGroups'].size()}
${parameters.regions.contains('us-east-1')}
${trigger?.buildInfo?.number ?: 'unknown'}
${parameters.env != null ? parameters.env : 'dev'}
${#readJson('{\"key\": \"value\"}')['key']}
${#stage('Webhook')['context']['webhook']['body']['status']}
```

## 🌐 Spinnaker REST API Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/applications` | GET | List all applications |
| `/applications/{app}` | GET | Get application details |
| `/applications/{app}/pipelines` | GET | List pipelines for application |
| `/applications/{app}/pipelineConfigs` | GET | Get pipeline configurations |
| `/pipelines/{id}` | GET | Get pipeline execution details |
| `/pipelines/{app}/{pipeline}` | POST | Trigger pipeline execution |
| `/pipelines/{id}/cancel` | PUT | Cancel running pipeline |
| `/credentials` | GET | List configured accounts |
| `/search` | GET | Search for resources |
| `/v2/canaryConfig` | GET/POST | Manage canary configurations |

```bash
curl http://gate:8084/applications

curl http://gate:8084/applications/myapp

curl http://gate:8084/applications/myapp/pipelineConfigs

curl -X POST http://gate:8084/pipelines/myapp/deploy-prod \
  -H "Content-Type: application/json" \
  -d '{
    "type": "manual",
    "parameters": {
      "version": "1.2.3"
    }
  }'

curl http://gate:8084/pipelines/{execution-id}

curl -X PUT http://gate:8084/pipelines/{execution-id}/cancel

curl http://gate:8084/credentials

curl "http://gate:8084/search?q=myapp&type=applications"

curl "http://gate:8084/applications/myapp/serverGroups"

curl "http://gate:8084/applications/myapp/loadBalancers"
```

## 🚀 Deployment Strategies Comparison

| Strategy | Downtime | Rollback Speed | Resource Cost | Best For |
|---|---|---|---|---|
| Blue/Green | Zero | Instant | High (2x resources) | Critical production apps |
| Canary | Zero | Medium | Medium | High-traffic applications |
| Rolling | Zero | Moderate | Minimal | Resource-constrained environments |
| Highlander | Zero | Slow | Very Low | Cost-sensitive deployments |
| Red/Black | Zero | Instant | High | AWS deployments |
| Custom | Varies | Varies | Varies | Specific requirements |

## 🔍 Troubleshooting Quick Reference

### Pods Not Starting

Symptoms: Pods stuck in Pending/CrashLoopBackOff

```bash
kubectl get pods -n spinnaker
kubectl describe pod {pod-name} -n spinnaker
kubectl logs {pod-name} -n spinnaker
```

Common Causes: Insufficient resources, image pull errors, config errors

### High Clouddriver Memory

Symptoms: OOMKilled, slow responses

```bash
kubectl top pod -n spinnaker | grep clouddriver

hal config provider aws edit \
  --cache-ttl-seconds 300

kubectl rollout restart deployment/clouddriver -n spinnaker

kubectl top pods -n spinnaker
```

Solutions: Increase memory, optimize caching, filter regions

### Pipeline Stuck

Symptoms: Pipeline not progressing

```bash
kubectl logs deployment/orca -n spinnaker

kubectl exec -it deployment/redis -n spinnaker -- redis-cli ping

curl -X PUT http://gate:8084/pipelines/{id}/cancel
```

Common Causes: Orca issues, Redis problems, stage timeouts

### Cannot Connect to Gate

Symptoms: UI/API unreachable

```bash
kubectl get svc gate -n spinnaker

kubectl exec -it deployment/deck -n spinnaker -- \
  curl http://gate:8084/health

kubectl logs deployment/gate -n spinnaker
```

Solutions: Verify service, check network policies, review configs

### Cloud Provider Not Listed

Symptoms: Accounts not appearing in UI

```bash
hal config provider aws account list

kubectl exec -it deployment/clouddriver -n spinnaker -- \
  curl localhost:7002/credentials

kubectl rollout restart deployment/clouddriver -n spinnaker
```

Solutions: Verify credentials, check IAM roles, restart Clouddriver

### Database Connection Failed

Symptoms: Front50 failing to start

```bash
kubectl logs deployment/front50 -n spinnaker

mysql -h {db-host} -u {user} -p

hal config storage
```

Solutions: Verify DB credentials, check network access, validate DB exists

## ✅ Best Practices Checklist

### Security Best Practices

- Enable RBAC with Fiat for access control
- Use service accounts for CI/CD integrations
- Never store secrets in pipeline configurations
- Enable TLS/SSL for all endpoints
- Implement audit logging for compliance
- Use Vault or cloud secret managers
- Apply principle of least privilege
- Regular security audits and reviews
- Multi-factor authentication for admins
- Network isolation between environments

### Operational Best Practices

- Deploy Spinnaker in HA configuration
- Use managed database services (RDS, CloudSQL)
- Implement automated backups daily
- Monitor all services with Prometheus/Grafana
- Set up alerting for critical failures
- Keep previous server groups for 2-4 hours
- Use infrastructure as code (Terraform)
- Document all custom configurations
- Regular disaster recovery testing
- Version control Halyard configurations

### Pipeline Design Best Practices

- Use pipeline templates for reusability
- Externalize configuration to parameter stores
- Implement automated testing at each stage
- Add manual approval for production
- Use canary deployments for high-risk changes
- Configure automated rollback on failure
- Add bake time after deployments
- Use pipeline expressions for dynamic config
- Implement notifications at key stages
- Keep pipelines modular and simple

### Performance Optimization

- Scale Clouddriver horizontally (4-6 replicas)
- Use Redis cluster for distributed caching
- Configure aggressive caching for stable resources
- Limit cloud accounts to necessary regions
- Use database read replicas
- Enable API rate limiting
- Clean up old pipeline executions (30-90 days)
- Optimize JVM heap sizes per service
- Use horizontal pod autoscaling
- Monitor and optimize slow queries

## 📊 Resource Sizing Guide

### Recommended Resource Allocation

| Service | Memory | CPU | Replicas | Notes |
|---|---|---|---|---|
| Clouddriver | 8 GB | 4 cores | 4-6 | Most resource-intensive |
| Orca | 4 GB | 2 cores | 3-5 | Scale based on pipeline volume |
| Gate | 2 GB | 1 core | 3-4 | Scale for API traffic |
| Deck | 1 GB | 0.5 cores | 2-3 | Lightweight UI service |
| Front50 | 2 GB | 1 core | 2-3 | Depends on app count |
| Echo | 2 GB | 1 core | 2-3 | Scale for notifications |
| Igor | 2 GB | 1 core | 2 | Based on CI integrations |
| Rosco | 4 GB | 2 cores | 2 | Scale for baking volume |
| Fiat | 2 GB | 1 core | 2 | RBAC overhead minimal |
| Kayenta | 4 GB | 2 cores | 2 | For canary analysis |
| Redis (cluster) | 8 GB | 2 cores | 3 | Use Redis cluster in prod |

## 🔗 Quick Links & Resources

**Official Documentation**
- Official Website — https://spinnaker.io
- Documentation — https://spinnaker.io/docs/
- Setup Guides — https://spinnaker.io/setup/
- Reference — https://spinnaker.io/reference/

**Community**
- Slack Community — https://join.spinnaker.io
- GitHub Organization — https://github.com/spinnaker
- Stack Overflow — https://stackoverflow.com/questions/tagged/spinnaker
- Community Hub — https://spinnaker.io/community/

**Learning Resources**
- YouTube Tutorials
- Spinnaker Summit Videos
- Community Webinars
- Blog Articles

**Tools & Plugins**
- Halyard (Config Tool)
- Spin CLI
- Kayenta (Canary Analysis)
- Plugin Repository

## 💬 Common Interview Questions — Quick Answers

**Q: What is Spinnaker?**
A: Open-source, multi-cloud continuous delivery platform that automates application deployments with advanced strategies like blue/green and canary.

**Q: Name the core Spinnaker microservices.**
A: Deck (UI), Gate (API gateway), Orca (orchestration), Clouddriver (cloud provider interface), Front50 (metadata storage), Echo (events), Igor (CI integration), Rosco (baking), Fiat (RBAC), Kayenta (canary analysis).

**Q: What deployment strategies does Spinnaker support?**
A: Blue/Green, Canary, Rolling, Red/Black, Highlander, and custom strategies.

**Q: How do you handle secrets in Spinnaker?**
A: Never store in pipelines. Use HashiCorp Vault, AWS Secrets Manager, or Kubernetes secrets with dynamic injection at runtime.

**Q: What is Halyard?**
A: Command-line tool for installing, configuring, and managing Spinnaker deployments.

**Q: How do you implement RBAC in Spinnaker?**
A: Using the Fiat service with LDAP/SAML integration, defining permissions per application/account, and implementing group-based access control.

---

> **Pro Tip:** Print this cheat sheet and keep it handy during development and troubleshooting sessions. Bookmark commonly used commands and API endpoints for quick access.

> **Remember:** Always test changes in non-production environments first. Validate Halyard configurations before deploying. Keep backups of all critical configurations.

---
*Source: adapted from the Spinnaker cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
