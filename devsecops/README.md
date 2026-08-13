# DevSecOps Cheatsheet

Quick reference guide for security-first development: scanning, container/Kubernetes security, CI/CD pipeline security, cloud security, secrets management, compliance, and incident response.

## 🔧 Essential DevSecOps Tools & Installation

### Security Scanning Tools

**Trivy - Container & Filesystem Scanner**

Vulnerability scanning for containers, filesystems, and git repositories.

```bash
brew install aquasecurity/trivy/trivy
apt-get install trivy

trivy image nginx:latest
trivy image --severity HIGH,CRITICAL nginx:latest
trivy fs /path/to/project
trivy image --format json --output report.json nginx:latest
trivy fs --scanners secret /path/to/project
```

**Snyk - Dependency Scanner**

Find and fix vulnerabilities in dependencies.

```bash
npm install -g snyk

snyk auth
snyk test
snyk container test nginx:latest
snyk monitor
snyk fix
```

**Gitleaks - Secret Scanner**

Detect hardcoded secrets in code.

```bash
brew install gitleaks

gitleaks detect --source . --verbose
gitleaks detect --report-path gitleaks-report.json
gitleaks detect --log-opts="--since=2024-01-01"
gitleaks protect --staged
```

**SonarQube - Code Quality & Security**

Static application security testing (SAST).

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest

sonar-scanner \
  -Dsonar.projectKey=my-project \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN

sonar-scanner -Dsonar.qualitygate.wait=true
```

### Infrastructure Security Tools

**Checkov - IaC Scanner**

Scan Terraform, CloudFormation, and Kubernetes manifests for misconfigurations.

```bash
pip install checkov

checkov -d /path/to/terraform
checkov -d /path/to/k8s --framework kubernetes
checkov -d . --check CKV_AWS_20,CKV_AWS_21
checkov -d . --output json > checkov-report.json
```

**tfsec - Terraform Security Scanner**

Security scanner specifically for Terraform.

```bash
brew install tfsec

tfsec .
tfsec . --minimum-severity HIGH
tfsec . --format json --out tfsec-report.json
```

## 🔍 Security Scanning Commands

### SAST (Static Application Security Testing)

```bash
pip install semgrep

semgrep --config=auto .
semgrep --config=p/owasp-top-ten .
semgrep --config=p/security-audit .
```

```bash
pip install bandit

bandit -r /path/to/code
bandit -r . -f json -o bandit-report.json
```

ESLint security plugin (JavaScript/TypeScript):

```bash
npm install --save-dev eslint eslint-plugin-security
eslint . --ext .js,.jsx,.ts,.tsx
```

Brakeman - Ruby on Rails security scanner:

```bash
gem install brakeman

brakeman -A -f json -o brakeman-report.json
```

### DAST (Dynamic Application Security Testing)

```bash
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://example.com \
  -r zap-report.html

nikto -h https://example.com -output nikto-report.html
nuclei -u https://example.com -severity high,critical
```

### SCA (Software Composition Analysis)

```bash
dependency-check.sh \
  --project "MyProject" \
  --scan /path/to/project \
  --format HTML \
  --out reports/

npm audit
npm audit --audit-level=high
npm audit fix

pip install pip-audit
pip-audit
pip-audit --requirement requirements.txt

bundle audit check --update
```

## 🐳 Container Security

### Docker Security Best Practices

```bash
docker build --no-cache --pull -t myapp:latest .
trivy image myapp:latest

docker run --rm \
  --read-only \
  --security-opt=no-new-privileges:true \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  -u 1000:1000 \
  myapp:latest

docker inspect --format='{{.HostConfig.SecurityOpt}}' container_name
docker history myapp:latest --no-trunc
```

### Dockerfile Security

```dockerfile
FROM python:3.11-slim AS base
RUN useradd -m -u 1000 appuser
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appuser . .
USER appuser
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8000/health || exit 1
EXPOSE 8000
CMD ["python", "app.py"]
```

**Docker Security Tips:**

- Always use specific image tags, never `:latest`
- Run containers as a non-root user
- Use multi-stage builds to reduce attack surface
- Scan images before deployment
- Enable Docker Content Trust for signed images
- Limit container resources (CPU, memory)

### Container Image Signing

```bash
cosign generate-key-pair
cosign sign --key cosign.key myapp:latest
cosign verify --key cosign.pub myapp:latest

export DOCKER_CONTENT_TRUST=1
docker push myapp:latest
docker pull myapp:latest
```

## ☸️ Kubernetes Security

### Pod Security Standards

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: myapp:latest
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
      resources:
        limits:
          cpu: "500m"
          memory: "512Mi"
        requests:
          cpu: "250m"
          memory: "256Mi"
      volumeMounts:
        - name: tmp
          mountPath: /tmp
  volumes:
    - name: tmp
      emptyDir: {}
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### Kubernetes Security Commands

```bash
kubectl auth can-i create pods --as=user@example.com
kubectl auth can-i --list

kubectl get pods -o json | jq '.items[].spec.securityContext'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].securityContext.privileged}{"\n"}{end}'

kubesec scan pod.yaml

helm install falco falcosecurity/falco --namespace falco

kubectl get secrets -o json | jq '.items[].data'

kubectl apply -f constraint-template.yaml
kubectl apply -f constraint.yaml
```

### Kubernetes Security Checklist

- [ ] Set `runAsNonRoot: true` in pod spec — run as non-root
- [ ] Set `readOnlyRootFilesystem: true` — read-only filesystem
- [ ] Drop ALL capabilities, add only what's needed
- [ ] Implement zero-trust networking with network policies
- [ ] Enforce least-privilege access control (RBAC)
- [ ] Enable encryption at rest for secrets
- [ ] Enforce restricted Pod Security Standards (PSS)
- [ ] Set CPU and memory resource limits

## 🔄 CI/CD Pipeline Security

### GitHub Actions Security

```yaml
# Secure GitHub Actions workflow
name: Secure CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
permissions:
  contents: read
  security-events: write
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'HIGH,CRITICAL'
          exit-code: '1'
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
      - name: SAST with Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/security-audit
      - name: Dependency check
        run: |
          npm audit --audit-level=high
          pip-audit
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Scan Docker image
        run: trivy image --severity HIGH,CRITICAL myapp:${{ github.sha }}
      - name: Sign image with Cosign
        run: |
          cosign sign --key ${{ secrets.COSIGN_KEY }} \
            myapp:${{ github.sha }}
```

### GitLab CI Security

```yaml
# GitLab CI pipeline with security scanning
stages:
  - security
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  SECURE_LOG_LEVEL: info

sast:
  stage: security
  image: returntocorp/semgrep:latest
  script:
    - semgrep --config=auto --json > sast-report.json
  artifacts:
    reports:
      sast: sast-report.json

dependency_scanning:
  stage: security
  image: python:3.11
  script:
    - pip install pip-audit
    - pip-audit --format json > dependency-report.json
  artifacts:
    reports:
      dependency_scanning: dependency-report.json

secret_detection:
  stage: security
  image: zricethezav/gitleaks:latest
  script:
    - gitleaks detect --source . --report-path gitleaks-report.json
  artifacts:
    paths:
      - gitleaks-report.json

container_scanning:
  stage: build
  image: aquasec/trivy:latest
  script:
    - trivy image --format json --output trivy-report.json $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  artifacts:
    reports:
      container_scanning: trivy-report.json
```

### Jenkins Pipeline Security

```groovy
pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'registry.company.com'
        SNYK_TOKEN = credentials('snyk-token')
    }
    stages {
        stage('Security Scan') {
            parallel {
                stage('SAST') {
                    steps {
                        sh 'semgrep --config=auto .'
                    }
                }
                stage('Secret Scan') {
                    steps {
                        sh 'gitleaks detect --source . --verbose'
                    }
                }
                stage('Dependency Check') {
                    steps {
                        sh 'snyk test --severity-threshold=high'
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/myapp:$BUILD_NUMBER .'
            }
        }
        stage('Container Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_REGISTRY/myapp:$BUILD_NUMBER'
            }
        }
        stage('Security Gate') {
            steps {
                script {
                    def scanResults = sh(returnStdout: true, script: 'trivy image --format json $DOCKER_REGISTRY/myapp:$BUILD_NUMBER')
                    def criticalVulns = sh(returnStdout: true, script: 'echo "$scanResults" | jq "[.Results[].Vulnerabilities[]? | select(.Severity == \\"CRITICAL\\")] | length"')
                    if (criticalVulns.toInteger() > 0) {
                        error("Critical vulnerabilities found. Blocking deployment.")
                    }
                }
            }
        }
    }
    post {
        always {
            publishHTML([
                reportDir: 'reports',
                reportFiles: 'security-report.html',
                reportName: 'Security Scan Report'
            ])
        }
    }
}
```

## ☁️ Cloud Security (AWS/Azure/GCP)

### AWS Security Commands

```bash
# Security audit
aws iam get-account-password-policy
aws iam list-users
aws iam get-user --user-name username
aws iam list-access-keys --user-name username

# Check for public S3 buckets
aws s3api list-buckets --query "Buckets[].Name" | xargs -I {} aws s3api get-bucket-acl --bucket {}

# Enable CloudTrail
aws cloudtrail create-trail --name security-audit-trail --s3-bucket-name audit-logs
aws cloudtrail start-logging --name security-audit-trail

# Enable GuardDuty
aws guardduty create-detector --enable

# Security Hub findings
aws securityhub get-findings --filters '{"SeverityLabel": [{"Value":"CRITICAL","Comparison":"EQUALS"}]}'

# Check EC2 security groups open to the world
aws ec2 describe-security-groups --query "SecurityGroups[?IpPermissions[?IpRanges[?CidrIp=='0.0.0.0/0']]]"

# Enable EBS encryption by default
aws ec2 enable-ebs-encryption-by-default
```

### Azure Security Commands

```bash
# Azure Security Center assessments and alerts
az security assessment list
az security alert list

# Enable auto-provisioning for the Azure security agent
az security auto-provisioning-setting update --name default --auto-provision on

# Check NSG rules
az network nsg list
az network nsg rule list --resource-group myResourceGroup --nsg-name myNSG

# Enable diagnostic logging
az monitor diagnostic-settings create \
  --resource /subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --name myDiagSetting \
  --logs '[{"category":"Administrative","enabled":true}]'

az keyvault list
az keyvault secret list --vault-name myKeyVault
az storage account list --query "[?publicNetworkAccess=='Enabled']"
```

### GCP Security Commands

```bash
gcloud projects get-iam-policy PROJECT_ID
gcloud compute firewall-rules list
gcloud services enable securitycenter.googleapis.com
gsutil iam get gs://BUCKET_NAME
gcloud logging read "resource.type=gce_instance" --limit 30
gcloud iam service-accounts list
gcloud iam service-accounts keys list --iam-account=SA_EMAIL
gcloud compute firewall-rules list --filter="direction=INGRESS AND allowed.ports:22"
```

## 🔑 Secrets Management

### HashiCorp Vault

```bash
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200'

# Write secret
vault kv put secret/myapp/config username=admin password=secret123

# Read secret
vault kv get secret/myapp/config
vault kv get -format=json secret/myapp/config | jq .data.data

# Enable audit logging
vault audit enable file file_path=/var/log/vault_audit.log

# Create policy
vault policy write myapp-policy - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
EOF

# Generate token with policy
vault token create -policy=myapp-policy

# Enable database secrets engine
vault secrets enable database
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/mydb"
```

### AWS Secrets Manager

```bash
aws secretsmanager create-secret --name prod/db/password --secret-string "MySecretPassword123!"
aws secretsmanager get-secret-value --secret-id prod/db/password --query SecretString --output text
aws secretsmanager rotate-secret --secret-id prod/db/password
aws secretsmanager delete-secret --secret-id prod/db/password --recovery-window-in-days 30
```

### Secrets Detection & Prevention

```bash
git secrets --install
git secrets --register-aws
git secrets --scan

trufflehog git https://github.com/user/repo --since-commit HEAD~10
trufflehog filesystem /path/to/files

detect-secrets scan > .secrets.baseline
detect-secrets audit .secrets.baseline
```

**Never Do This:**

- Hard-code credentials in source code
- Commit secrets to version control
- Store secrets in container images
- Share secrets via email or chat
- Use the same password across environments
- Leave default credentials unchanged

## 📋 Compliance & Auditing

### Compliance Frameworks

| Framework | Focus Areas | Example Tools |
|---|---|---|
| SOC 2 | Access control, encryption, monitoring, incident response | Vanta, Drata, SecureFrame |
| PCI-DSS | Network segmentation, encryption, access control, logging | Qualys, Trustwave, SecurityMetrics |
| HIPAA | Data encryption, audit controls, access management | Vanta, Compliancy Group |
| GDPR | Data privacy, consent, right to erasure, data portability | OneTrust, TrustArc |
| ISO 27001 | ISMS, risk assessment, security controls | Compliance.ai, Secureframe |

### Audit Logging Commands

```bash
# Linux audit
auditctl -l
auditctl -w /etc/passwd -p wa -k passwd_changes
ausearch -k passwd_changes

# Docker logs
docker logs container_name
docker logs --since 1h container_name
docker logs -f container_name

# Kubernetes logs
kubectl logs -n kube-system kube-apiserver-master
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Git history
git log --all --oneline --graph
git log --author="username" --since="2024-01-01"
git log --follow -- filename
```

### Compliance Automation with OPA

```rego
# OPA Rego policy for compliance
package compliance.pci_dss

# PCI-DSS Requirement: encrypt cardholder data at rest
deny[msg] {
  input.kind == "PersistentVolumeClaim"
  not input.metadata.annotations["encrypted"] == "true"
  msg := sprintf("PVC %s must be encrypted", [input.metadata.name])
}

# PCI-DSS Requirement: network segmentation
deny[msg] {
  input.kind == "Namespace"
  input.metadata.labels["cardholder-data"] == "true"
  not has_network_policy(input.metadata.name)
  msg := sprintf("Namespace %s with cardholder data must have NetworkPolicy", [input.metadata.name])
}
```

```bash
opa eval --data policy.rego --input manifest.yaml "data.compliance.pci_dss.deny"
```

## 🚨 Incident Response

### Incident Response Playbook

**1. Detection & Identification**

```bash
ps aux | grep -E 'nc|ncat|netcat|/dev/tcp'
netstat -tulpn | grep ESTABLISHED
ss -tunap
find / -type f -mtime -2 2>/dev/null | grep -v '/proc\|/sys'
lastlog
last -f /var/log/wtmp
grep "Failed password" /var/log/auth.log
```

**2. Containment**

```bash
# Isolate compromised container
kubectl label pod compromised-pod quarantine=true
kubectl patch networkpolicy default --patch '{"spec":{"podSelector":{"matchLabels":{"quarantine":"true"}},"policyTypes":["Ingress","Egress"]}}'

# Block malicious IP address
iptables -A INPUT -s MALICIOUS_IP -j DROP
aws ec2 revoke-security-group-ingress --group-id sg-xxx --ip-permissions \
  IpProtocol=tcp,FromPort=0,ToPort=65535,IpRanges='[{CidrIp=MALICIOUS_IP/32}]'

aws iam delete-access-key --user-name compromised-user --access-key-id AKIAIOSFODNN7EXAMPLE
kubectl delete serviceaccount compromised-sa
```

**3. Forensics & Evidence Collection**

```bash
# Capture container state
docker inspect container_id > container-forensics.json
docker logs container_id > container-logs.txt
docker exec container_id ps aux > container-processes.txt

# Capture network traffic
tcpdump -i eth0 -w capture.pcap

# Memory dump
gcore PID

# Disk image
dd if=/dev/sda of=/mnt/backup/disk-image.img bs=4M

# Kubernetes events
kubectl get events --all-namespaces --sort-by='.lastTimestamp' > k8s-events.txt
```

**4. Recovery**

```bash
kubectl delete pod compromised-pod
kubectl apply -f clean-deployment.yaml
vault write -f sys/rotate
aws secretsmanager rotate-secret --secret-id compromised-secret
kubectl apply -f updated-network-policy.yaml
kubectl apply -f updated-pod-security-policy.yaml
```

## ✅ Best Practices & Quick Tips

### Authentication

- Enforce MFA for all accounts
- Use certificate-based authentication
- Implement short-lived tokens (1-24 hours)
- Rotate credentials regularly
- Use SSH keys, never passwords

### Authorization

- Principle of least privilege
- Role-based access control (RBAC)
- Just-in-time access provisioning
- Regular access reviews
- Separate dev/staging/prod access

### Encryption

- TLS 1.2+ for all traffic
- Encrypt data at rest (AES-256)
- Encrypt backups
- Use managed encryption keys
- Rotate encryption keys annually

### Monitoring

- Centralized logging (ELK, Splunk)
- Real-time alerting
- Security metrics dashboard
- Anomaly detection with ML
- Log retention of 90+ days

### CI/CD Security

- Scan code on every commit
- Automated security gates
- Sign and verify artifacts
- Immutable build environments
- Secure pipeline credentials

### Container Security

- Use minimal base images
- Run as a non-root user
- Scan images before deployment
- Regular image updates
- Remove unnecessary tools

### Cloud Security

- Enable all audit logging
- Use cloud-native security tools
- Implement network segmentation
- Regular security assessments
- Automate compliance checks

### Documentation

- Document security architecture
- Maintain runbooks
- Maintain security incident playbooks
- Regular security training
- Update documentation quarterly

### Security Metrics to Track

| Metric | Description | Target |
|---|---|---|
| Mean Time to Detect (MTTD) | Time from breach to detection | < 1 hour |
| Mean Time to Respond (MTTR) | Time from detection to containment | < 4 hours |
| Vulnerability Remediation Time | Time from discovery to fix | < 7 days (critical) |
| Security Scan Coverage | % of deployments scanned | 100% |
| Critical Vulnerabilities | Count of CVSS 9.0+ vulns | 0 in production |
| Failed Security Tests | % of builds failing security gates | < 5% |
| Secrets Detected | Count from secret scanners | 0 |
| Security Training Completion | % of team trained annually | 100% |

### Pro Tips for DevSecOps Success

- **Shift Left**: Integrate security early in development, not at the end
- **Automate Everything**: Manual security processes don't scale
- **Fail Fast**: Block insecure code before it reaches production
- **Measure Progress**: Track security metrics over time
- **Culture Matters**: Make security everyone's responsibility
- **Learn Continuously**: The security landscape changes rapidly
- **Test Your Defenses**: Run regular penetration testing and red team exercises
- **Embrace Chaos**: Practice incident response with chaos engineering

### Common Pitfalls to Avoid

- Treating security as an afterthought
- Ignoring low/medium vulnerabilities (they add up!)
- No security champions in development teams
- Manual security reviews as bottlenecks
- Alert fatigue from too many false positives
- Lack of security training for developers
- No incident response plan
- Security tools without integration

---

*Source: adapted from the DevSecOps cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
