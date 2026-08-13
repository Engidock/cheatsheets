# Helm Cheatsheet

Quick reference for the Kubernetes package manager.

## 🔧 Installation & Setup

### Install Helm

```bash
# Linux/macOS - Script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# macOS - Homebrew
brew install helm

# Windows - Chocolatey
choco install kubernetes-helm

# Windows - Scoop
scoop install helm

# Verify installation
helm version
```

### Configure Repositories

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo add nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update repositories
helm repo update

# List repositories
helm repo list

# Remove repository
helm repo remove bitnami
```

### Shell Completion

```bash
# Bash
source <(helm completion bash)
echo "source <(helm completion bash)" >> ~/.bashrc

# Zsh
source <(helm completion zsh)
echo "source <(helm completion zsh)" >> ~/.zshrc
```

## 📦 Chart Management

### Search Charts

```bash
# Search in repositories
helm search repo nginx
helm search repo database

# Search all versions
helm search repo nginx --versions

# Search Artifact Hub
helm search hub wordpress
```

### Chart Information

```bash
# Show chart details
helm show chart bitnami/nginx
helm show values bitnami/nginx
helm show readme bitnami/nginx
helm show all bitnami/nginx

# Pull chart
helm pull bitnami/nginx
helm pull bitnami/nginx --version 15.0.0
helm pull bitnami/nginx --untar
helm pull bitnami/nginx --destination ./charts/
```

### Create Chart

```bash
# Create new chart
helm create mychart

# Validate chart
helm lint mychart

# Package chart
helm package mychart

# Package with version
helm package mychart --version 1.2.3

# Package with app version
helm package mychart --app-version v1.0.0
```

## 🚀 Release Management

### Install Release

```bash
# Basic install
helm install myrelease bitnami/nginx

# Install in namespace
helm install myrelease bitnami/nginx -n production

# Create namespace
helm install myrelease bitnami/nginx -n production --create-namespace

# Install with values file
helm install myrelease bitnami/nginx -f values.yaml

# Install with inline values
helm install myrelease bitnami/nginx --set replicaCount=3

# Install specific version
helm install myrelease bitnami/nginx --version 15.0.0

# Dry-run
helm install myrelease bitnami/nginx --dry-run --debug

# Wait for resources
helm install myrelease bitnami/nginx --wait --timeout 10m

# Generate name
helm install bitnami/nginx --generate-name
```

### List Releases

```bash
# List releases
helm list
helm list -n production
helm list --all-namespaces
helm list -A

# List all (including failed)
helm list --all

# Filter by status
helm list --deployed
helm list --failed
helm list --pending

# Output formats
helm list -o json
helm list -o yaml
```

### Release Status & History

```bash
# Get release status
helm status myrelease
helm status myrelease -n production

# Get release history
helm history myrelease
helm history myrelease -n production

# Get release values
helm get values myrelease
helm get values myrelease --all

# Get release manifest
helm get manifest myrelease

# Get release notes
helm get notes myrelease

# Get all release info
helm get all myrelease
```

### Upgrade Release

```bash
# Basic upgrade
helm upgrade myrelease bitnami/nginx

# Upgrade with values
helm upgrade myrelease bitnami/nginx -f values.yaml

# Upgrade with new values
helm upgrade myrelease bitnami/nginx --set image.tag=1.22

# Reuse existing values
helm upgrade myrelease bitnami/nginx --reuse-values

# Reset to chart defaults
helm upgrade myrelease bitnami/nginx --reset-values

# Force upgrade
helm upgrade myrelease bitnami/nginx --force

# Dry-run upgrade
helm upgrade myrelease bitnami/nginx --dry-run --debug

# Install if not exists
helm upgrade --install myrelease bitnami/nginx

# Wait for upgrade
helm upgrade myrelease bitnami/nginx --wait --timeout 10m
```

### Rollback Release

```bash
# Rollback to previous
helm rollback myrelease

# Rollback to specific revision
helm rollback myrelease 2

# Dry-run rollback
helm rollback myrelease --dry-run

# Wait for rollback
helm rollback myrelease --wait
```

### Uninstall Release

```bash
# Uninstall release
helm uninstall myrelease
helm uninstall myrelease -n production

# Keep history
helm uninstall myrelease --keep-history

# Dry-run uninstall
helm uninstall myrelease --dry-run
```

### Test Release

```bash
# Run tests
helm test myrelease
helm test myrelease -n production

# Show logs
helm test myrelease --logs
```

## 📝 Template Commands

### Render Templates

```bash
# Render templates
helm template myrelease mychart

# With values file
helm template myrelease mychart -f values.yaml

# With inline values
helm template myrelease mychart --set replicaCount=3

# Render specific template
helm template myrelease mychart -s templates/deployment.yaml

# Show only manifests
helm template myrelease mychart --show-only templates/deployment.yaml

# Include CRDs
helm template myrelease mychart --include-crds

# Debug mode
helm template myrelease mychart --debug
```

### Validate Templates

```bash
# Validate against Kubernetes
helm template myrelease mychart | kubectl apply --dry-run=client -f -

# Lint chart
helm lint mychart

# Lint with values
helm lint mychart -f values.yaml
```

## 🛠️ Chart Development

### Chart Structure

```
mychart/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default values
├── charts/                 # Dependency charts
├── templates/               # Template files
│   ├── NOTES.txt           # Post-install notes
│   ├── _helpers.tpl        # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── tests/
│       └── test-connection.yaml
├── .helmignore              # Files to ignore
└── README.md                # Chart documentation
```

### Chart.yaml

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
type: application
version: 1.0.0
appVersion: "1.0"
keywords:
  - web
  - nginx
home: https://github.com/myorg/mychart
sources:
  - https://github.com/myorg/mychart
maintainers:
  - name: Your Name
    email: you@example.com
dependencies:
  - name: mysql
    version: 9.3.4
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
```

### values.yaml

```yaml
replicaCount: 1
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
nodeSelector: {}
tolerations: []
affinity: {}
```

### Common Template Functions

```gotemplate
# String functions
{{ .Values.name | upper }}
{{ .Values.name | lower }}
{{ .Values.name | quote }}
{{ .Values.name | trim }}
{{ .Values.name | default "default-value" }}
{{ .Values.name | required "name is required" }}

# Conversion functions
{{ .Values.data | toYaml }}
{{ .Values.data | toJson }}
{{ .Values.data | toString }}
{{ .Values.number | int }}

# Indentation
{{ .Values.data | nindent 4 }}
{{ .Values.data | indent 2 }}

# List functions
{{ .Values.list | join "," }}
{{ .Values.list | first }}
{{ .Values.list | last }}

# Conditionals
{{ if .Values.enabled }}
  enabled: true
{{ else }}
  enabled: false
{{ end }}

# Range (loops)
{{ range .Values.items }}
- {{ . }}
{{ end }}

# Include named templates
{{ include "mychart.fullname" . }}
{{ include "mychart.labels" . | nindent 4 }}
```

### Helper Templates (_helpers.tpl)

```gotemplate
{{/*
Chart name
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Fullname
*/}}
{{- define "mychart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Chart label
*/}}
{{- define "mychart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### Deployment Template Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "mychart.serviceAccountName" . }}
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.port }}
          protocol: TCP
        {{- if .Values.livenessProbe }}
        livenessProbe:
          {{- toYaml .Values.livenessProbe | nindent 10 }}
        {{- end }}
        {{- if .Values.readinessProbe }}
        readinessProbe:
          {{- toYaml .Values.readinessProbe | nindent 10 }}
        {{- end }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### Hooks

```yaml
# Pre-install hook
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mychart.fullname" . }}-init
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: init
        image: busybox
        command: ['sh', '-c', 'echo Initializing...']
      restartPolicy: Never
```

```text
# Available hooks:
# pre-install, post-install
# pre-upgrade, post-upgrade
# pre-delete, post-delete
# pre-rollback, post-rollback
# Hook weights: -5, 0, 5 (lower runs first)
# Delete policies:
# before-hook-creation, hook-succeeded, hook-failed
```

### Tests

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

## 🔗 Dependencies

### Define Dependencies

```yaml
# Chart.yaml
dependencies:
  - name: mysql
    version: 9.3.4
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
  - name: redis
    version: 17.3.7
    repository: https://charts.bitnami.com/bitnami
    alias: cache
  - name: common
    version: 1.0.0
    repository: "file://../common"
```

### Manage Dependencies

```bash
# Download dependencies
helm dependency update mychart

# List dependencies
helm dependency list mychart

# Build dependencies
helm dependency build mychart
```

### Configure Subcharts

```yaml
# values.yaml
global:
  storageClass: "fast"
mysql:
  enabled: true
  auth:
    rootPassword: "password"
    database: "myapp"
  primary:
    persistence:
      enabled: true
      size: 8Gi
redis:
  enabled: false
```

## ⚙️ Advanced Commands

### Helm Plugins

```bash
# List plugins
helm plugin list

# Install plugin
helm plugin install https://github.com/databus23/helm-diff

# Uninstall plugin
helm plugin uninstall diff

# Update plugin
helm plugin update diff
```

### Useful Plugins

```bash
# helm-diff - Preview upgrade changes
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade myrelease mychart -f values.yaml

# helm-secrets - Encrypt secrets
helm plugin install https://github.com/jkroepke/helm-secrets
helm secrets install myrelease mychart -f secrets.enc.yaml

# helm-git - Use Git repositories
helm plugin install https://github.com/aslafy-z/helm-git
helm install myrelease git+https://github.com/org/repo@charts/mychart
```

### Environment & Config

```bash
# Show environment
helm env

# Get config
helm get values myrelease --all

# Set environment variables
export HELM_NAMESPACE=production
export HELM_DEBUG=true
export HELM_KUBECONTEXT=my-cluster
```

### OCI Registry

```bash
# Login to OCI registry
helm registry login registry.example.com -u user -p password

# Push chart to OCI
helm push mychart-1.0.0.tgz oci://registry.example.com/helm-charts

# Install from OCI
helm install myrelease oci://registry.example.com/helm-charts/mychart --version 1.0.0

# Pull from OCI
helm pull oci://registry.example.com/helm-charts/mychart --version 1.0.0
```

### Repository Index

```bash
# Create repository index
helm repo index . --url https://charts.example.com

# Merge with existing index
helm repo index . --url https://charts.example.com --merge index.yaml
```

## 🎯 Common Patterns & Best Practices

### Values Override Patterns

```bash
# Multiple values files (order matters)
helm install myrelease mychart \
  -f values.yaml \
  -f values-production.yaml \
  -f values-override.yaml

# Environment-specific values
helm install myrelease mychart -f values-${ENV}.yaml

# Set nested values
helm install myrelease mychart \
  --set image.repository=myrepo/myimage \
  --set image.tag=v1.2.3

# Set array values
helm install myrelease mychart \
  --set-string env[0].name=KEY1 \
  --set-string env[0].value=value1 \
  --set-string env[1].name=KEY2 \
  --set-string env[1].value=value2

# Set from file
helm install myrelease mychart \
  --set-file config=./config.txt
```

### Conditional Resources

```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "mychart.fullname" . }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "mychart.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

### ConfigMap from Files

```
# Directory structure
mychart/
├── templates/
│   └── configmap.yaml
└── config/
    ├── app.conf
    └── settings.json
```

```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mychart.fullname" . }}
data:
  {{- (.Files.Glob "config/*").AsConfig | nindent 2 }}

# Or specific files
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mychart.fullname" . }}
data:
  app.conf: |
{{ .Files.Get "config/app.conf" | indent 4 }}
  settings.json: |
{{ .Files.Get "config/settings.json" | indent 4 }}
```

### Security Context

```yaml
# values.yaml
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
```

```yaml
# In template
spec:
  securityContext:
    {{- toYaml .Values.podSecurityContext | nindent 8 }}
  containers:
  - name: {{ .Chart.Name }}
    securityContext:
      {{- toYaml .Values.securityContext | nindent 12 }}
```

### Resource Limits

```yaml
# values.yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

```yaml
# In template
{{- if .Values.resources }}
resources:
  {{- toYaml .Values.resources | nindent 10 }}
{{- end }}
```

### Health Probes

```yaml
# values.yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
```

```yaml
# In template
livenessProbe:
  {{- toYaml .Values.livenessProbe | nindent 10 }}
readinessProbe:
  {{- toYaml .Values.readinessProbe | nindent 10 }}
```

## 🔍 Troubleshooting

### Debug Commands

```bash
# Enable debug mode
helm install myrelease mychart --debug --dry-run

# Verbose output
helm install myrelease mychart -v 4

# Render templates
helm template myrelease mychart --debug

# Get release manifest
helm get manifest myrelease

# Check values
helm get values myrelease --all

# List all resources
kubectl get all -l app.kubernetes.io/instance=myrelease
```

### Common Issues

```bash
# Clear cache
rm -rf ~/.cache/helm/*

# Force update repo
helm repo update --fail-on-repo-update-fail

# Fix stuck release
kubectl delete secret -n default sh.helm.release.v1.myrelease.v1

# Check pending operations
helm list --pending

# Rollback failed upgrade
helm rollback myrelease 0
```

### Validation

```bash
# Validate chart
helm lint mychart

# Validate against Kubernetes
helm template myrelease mychart | kubectl apply --dry-run=client -f -

# Check rendered output
helm template myrelease mychart > output.yaml
cat output.yaml
```

## 📊 Quick Reference Tables

### Common Flags

| Flag | Description |
|---|---|
| `-n, --namespace` | Kubernetes namespace |
| `-f, --values` | Values file |
| `--set` | Set values inline |
| `--version` | Chart version |
| `--wait` | Wait for resources to be ready |
| `--timeout` | Time to wait (default 5m) |
| `--dry-run` | Simulate install/upgrade |
| `--debug` | Enable debug output |
| `--create-namespace` | Create namespace if not exists |
| `--force` | Force resource updates |

### Built-in Objects

| Object | Description |
|---|---|
| `.Release.Name` | Release name |
| `.Release.Namespace` | Release namespace |
| `.Release.Service` | Service (always "Helm") |
| `.Release.IsUpgrade` | True if upgrade |
| `.Release.IsInstall` | True if install |
| `.Chart.Name` | Chart name |
| `.Chart.Version` | Chart version |
| `.Chart.AppVersion` | Application version |
| `.Values` | Values from values.yaml |
| `.Files` | Access to files in chart |
| `.Capabilities` | Kubernetes capabilities |
| `.Template` | Current template info |

### Template Functions

| Function | Example | Result |
|---|---|---|
| `quote` | `{{ .Values.name | quote }}` | `"value"` |
| `default` | `{{ .Values.name | default "nginx" }}` | `nginx` (if empty) |
| `upper` | `{{ .Values.name | upper }}` | `NGINX` |
| `lower` | `{{ .Values.name | lower }}` | `nginx` |
| `trim` | `{{ .Values.name | trim }}` | `nginx` |
| `nindent` | `{{ .Values.data | nindent 4 }}` | Newline + 4 spaces |
| `toYaml` | `{{ .Values.data | toYaml }}` | YAML format |
| `toJson` | `{{ .Values.data | toJson }}` | JSON format |

### Release Status Values

| Status | Description |
|---|---|
| `deployed` | Successfully deployed |
| `failed` | Deployment failed |
| `pending-install` | Install in progress |
| `pending-upgrade` | Upgrade in progress |
| `pending-rollback` | Rollback in progress |
| `superseded` | Replaced by newer release |
| `uninstalled` | Release uninstalled |
| `uninstalling` | Uninstall in progress |

## 💡 Pro Tips

**🎯 Best Practices:**
- Always use `--dry-run` before production deployments
- Version your charts with semantic versioning
- Use `--wait` to ensure resources are ready
- Store values files in version control
- Use separate values files for different environments
- Define resource limits for all containers
- Use liveness and readiness probes
- Always set security contexts
- Use `required` for mandatory values
- Document your charts with README and NOTES.txt

**⚠️ Common Pitfalls:**
- Don't hardcode values in templates
- Avoid using `latest` image tags
- Don't skip version control for charts
- Never commit secrets to Git
- Don't ignore resource limits
- Avoid deep nesting in values.yaml
- Don't forget to update dependencies
- Test upgrades in staging first

**📚 Resources:**
- Official Docs: https://helm.sh/docs/
- Chart Best Practices: https://helm.sh/docs/chart_best_practices/
- Artifact Hub: https://artifacthub.io/
- Helm GitHub: https://github.com/helm/helm

---
*Source: adapted from the Helm cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
