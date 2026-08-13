# Azure DevOps Cheatsheet

Architecting, building, and deploying with Microsoft's DevOps platform.

## I. Fundamentals & CLI

### Azure DevOps CLI (`az devops`)

The CLI is essential for automation and quick interactions without the UI.

#### 1. Installation & Login

Install extension.

```bash
az extension add --name azure-devops
```

Login with a Personal Access Token (PAT).

```bash
echo $MY_PAT | az devops login
```

Configure defaults (saves typing later).

```bash
az devops configure --defaults organization=https://dev.azure.com/myorg project=myproj
```

#### 2. Pipeline Operations

List all pipelines.

```bash
az pipelines list --output table
```

Run a pipeline (queue a build).

```bash
az pipelines run \
  --name "My-Microservice-CI" \
  --branch "feature/login" \
  --variables "debug=true"
```

Get the latest build ID.

```bash
az pipelines build list --query "[0].id" --output tsv
```

#### 3. Repository Operations

Create a Pull Request.

```bash
az repos pr create \
  --source-branch "feature/api" \
  --target-branch "main" \
  --title "Update API Endpoints" \
  --description "Fixes AB#123" \
  --open
```

List active PRs.

```bash
az repos pr list --status active --output table
```

## II. Azure Boards & Tracking

### Process Models & Hierarchies

Choosing the right process determines which Work Item Types (WITs) are available.

| Process | Hierarchy (Top-Down) | State Flow |
|---|---|---|
| Basic | Epic → Issue → Task | To Do → Doing → Done |
| Agile | Epic → Feature → User Story → Task | New → Active → Resolved → Closed |
| Scrum | Epic → Feature → PBI → Task | New → Approved → Committed → Done |
| CMMI | Epic → Feature → Requirement → Task | Proposed → Active → Resolved → Closed |

### WIQL (Work Item Query Language)

WIQL is SQL-like syntax used to power Dashboard Widgets and custom queries.

Find stale bugs.

```sql
SELECT [System.Id], [System.Title], [System.AssignedTo]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [System.State] <> 'Closed'
  AND [System.CreatedDate] < @Today - 30
ORDER BY [System.Priority] ASC
```

Find my tasks in the current sprint.

```sql
SELECT [System.Id], [System.Title], [System.State]
FROM WorkItems
WHERE [System.TeamProject] = @project
  AND [System.WorkItemType] = 'Task'
  AND [System.State] = 'Active'
  AND [System.AssignedTo] = @me
  AND [System.IterationPath] = @currentIteration
```

## III. Azure Repos & Policies

### Branch Policies (Best Practices)

Branch policies protect critical branches (`main`, `develop`) from direct pushes and bad code.

| Policy Type | Recommended Setting | Why? |
|---|---|---|
| Minimum Reviewers | At least 2 | Ensures code is reviewed by peers (four eyes principle). |
| Check for Linked Items | Required | Enforces traceability (why was this code changed?). |
| Build Validation | Enabled (CI Pipeline) | Ensures the code compiles and tests pass before merge. |
| Comment Resolution | Required | Ensures all feedback is addressed or resolved. |
| Limit Merge Types | Squash Merge | Keeps the history linear and clean. |

## IV. Pipelines: YAML Anatomy

### The Complete Pipeline Skeleton

```yaml
name: $(Date:yyyyMMdd).$(Rev:r) # 20231025.1
trigger: # Continuous Integration (CI) Trigger
  batch: true
  branches:
    include: [main, releases/*]
  paths:
    exclude: [docs/, README.md]
pr: # Pull Request Trigger
  branches:
    include: [main]
pool:
  vmImage: 'ubuntu-latest'
variables:
  buildConfig: 'Release'
  group: 'GlobalVars' # Variable Group
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - script: echo "Building..."
          - task: PublishBuildArtifacts@1
```

### Variables & Expressions Reference

| Syntax | Type | Time | Use Case |
|---|---|---|---|
| `$(myVar)` | Macro | Runtime | Standard access. Replaced before task runs. |
| `${{ variables.myVar }}` | Template | Compile | Conditionals for inserting steps/jobs. Loops. |
| `$[ variables.myVar ]` | Runtime | Runtime | Conditions on jobs/stages status. |

Example: conditional insertion (compile time).

```yaml
steps:
- ${{ if eq(parameters.environment, 'prod') }}:
  - script: echo "Running Security Scan..."
    displayName: Security Scan
```

## V. Advanced Pipeline Patterns

### Matrix Strategy

Run the same job configuration across multiple dimensions (OS, Node version) simultaneously.

```yaml
jobs:
- job: Test
  strategy:
    maxParallel: 3
    matrix:
      linux_node14:
        imageName: 'ubuntu-latest'
        nodeVersion: '14.x'
      windows_node16:
        imageName: 'windows-latest'
        nodeVersion: '16.x'
  pool:
    vmImage: $(imageName)
  steps:
  - task: NodeTool@0
    inputs:
      versionSpec: $(nodeVersion)
  - script: npm install && npm test
```

### Templates (Reuse)

Use `extends` to enforce security scanning and logging across all pipelines.

```yaml
# templates/secure-build.yml
parameters:
  - name: buildSteps
    type: stepList
    default: []
steps:
- task: CredScan@2 # Enforced Step
- ${{ parameters.buildSteps }} # Injected Steps
- task: PublishSymbols@2 # Enforced Step
```

```yaml
# azure-pipelines.yml
extends:
  template: templates/secure-build.yml
  parameters:
    buildSteps:
      - script: npm build
```

### Deployment Strategies (Canary)

```yaml
jobs:
- deployment: DeployCanary
  environment: 'production'
  strategy:
    canary:
      increments: [10, 20] # Traffic %
      preDeploy:
        steps:
        - script: ./init.sh
      deploy:
        steps:
        - task: KubernetesManifest@0
          inputs:
            action: $(strategy.action)
            manifests: manifests/deployment.yaml
```

## VI. Azure Artifacts

### Authentication Tasks

Required to restore packages from private feeds in Azure Artifacts.

NuGet (.NET).

```yaml
- task: NuGetAuthenticate@0
  displayName: 'Auth NuGet'
- script: dotnet restore --configfile nuget.config
```

npm (Node).

```yaml
- task: npmAuthenticate@0
  inputs:
    workingFile: .npmrc
```

Python (Twine).

```yaml
- task: TwineAuthenticate@1
  inputs:
    artifactFeed: 'MyFeed'
```

## VII. REST API Reference

### Curl Examples

Auth header: `Authorization: Basic base64(user:PAT)`

Get build details.

```bash
curl -u user:$PAT \
  "https://dev.azure.com/{org}/{proj}/_apis/build/builds/123?api-version=6.0"
```

Trigger a pipeline run.

```bash
curl -X POST -u user:$PAT \
  -H "Content-Type: application/json" \
  -d '{
        "definition": { "id": 15 },
        "sourceBranch": "refs/heads/main",
        "parameters": "{\"myVar\":\"value\"}"
      }' \
  "https://dev.azure.com/{org}/{proj}/_apis/build/builds?api-version=6.0"
```

Get a work item.

```bash
curl -u user:$PAT \
  "https://dev.azure.com/{org}/{proj}/_apis/wit/workitems/500?api-version=6.0"
```

## VIII. Troubleshooting Guide

### Common Pipeline Errors

| Error Message | Root Cause | Solution |
|---|---|---|
| YAML syntax error at line X | Indentation (tabs vs spaces). | Use a linter. Ensure 2-space indentation. No tabs allowed. |
| 403 Forbidden (Artifacts) | Build Service lacks permission. | Artifacts > Feed Settings > Permissions > Add "Project Collection Build Service" as Contributor. |
| Task not authorized | Service Connection security. | Project Settings > Service Connections > [Connection] > Security > "Grant access to all pipelines". |
| NU1101: Unable to find package | NuGet config missing feed. | Ensure `nuget.config` includes the private Azure Artifacts feed URL. |
| Agent pool not found | Pool name typo or agent offline. | Check `pool: name: 'MyPool'` matches exactly. Verify the self-hosted agent service is running. |

---
*Source: adapted from the Azure DevOps cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
