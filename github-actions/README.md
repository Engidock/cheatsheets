# Github Actions Cheatsheet

Complete reference guide for GitHub Actions workflows, jobs, actions, CI/CD automation, and best practices.

## 1. Fundamentals & Setup

GitHub Actions is a CI/CD platform that automates tasks in your GitHub workflow. Triggered by GitHub events (push, PR, schedule), it runs jobs in containers or VMs.

**Key Components:**

- **Workflow** - YAML file defining the automation
- **Event** - What triggers the workflow (push, pull_request, schedule)
- **Job** - Set of steps running on the same runner
- **Step** - Individual task within a job
- **Action** - Reusable unit of code
- **Runner** - Server executing the workflow

Repository Setup:

```bash
# Create workflows directory
mkdir -p .github/workflows

# Create first workflow file
touch .github/workflows/main.yml

# Push to GitHub
git add .github/workflows/main.yml
git commit -m "Add GitHub Actions workflow"
git push origin main
```

**System Requirements:**

- GitHub repository (public or private)
- GitHub account with write access
- Basic understanding of YAML
- No installation needed - cloud-based

GitHub CLI Installation:

```bash
# macOS
brew install gh

# Windows
choco install gh

# Linux
sudo apt install gh

# Authenticate
gh auth login
```

## 2. Workflow Basics & Syntax

Workflow File Structure:

```yaml
name: CI Pipeline # Workflow name
on: # Trigger events
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs: # Jobs to run
  test: # Job name
    runs-on: ubuntu-latest # Runner
    steps: # Steps in job
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
```

**Essential Syntax Elements:**

Workflow name:

```yaml
name: Deploy App
```

Trigger events:

```yaml
on: [push, pull_request]
```

Jobs to execute:

```yaml
jobs:
  build:
    ...
  test:
    ...
```

Runner type:

```yaml
runs-on: ubuntu-latest
```

Steps in job:

```yaml
steps:
  - run: npm test
```

## 3. Events & Triggers

**Push Event / Pull Request Event:**

```yaml
on: push
```

```yaml
on: pull_request
```

Triggered on code push / Triggered on PR creation/update:

```yaml
on:
  push:
    branches: [main]
    paths: ['src/**']

on:
  pull_request:
    branches: [main]
    types: [opened, edited]
```

**Schedule / Workflow Dispatch:**

```yaml
on: schedule
```

```yaml
on: workflow_dispatch
```

Cron-based scheduling / Manual trigger from UI:

```yaml
on:
  schedule:
    - cron: '0 9 * * 1'

on:
  workflow_dispatch:
    inputs:
      version:
        required: true
```

**Repository Dispatch / Release:**

```yaml
on: repository_dispatch
```

```yaml
on: release
```

External webhook trigger / Triggered on release:

```yaml
on:
  repository_dispatch:
    types: [deploy]

on:
  release:
    types: [published]
```

**Event Filtering:**

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
      - '!releases/beta'

on:
  push:
    paths:
      - 'src/**'
      - 'package.json'
      - '!docs/**'
```

## 4. Jobs & Steps Configuration

Job Configuration:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    strategy:
      matrix:
        node-version: [14, 16, 18]
      max-parallel: 3
    environment: production
    outputs:
      build-version: ${{ steps.version.outputs.version }}
    steps:
      - uses: actions/checkout@v3
      - name: Test
        run: npm test
```

**Job Options:**

Runner type (`runs-on`):

```yaml
runs-on: ubuntu-latest # or windows-latest, macos-latest
```

Job timeout, default 360 minutes (`timeout-minutes`):

```yaml
timeout-minutes: 30
```

Deployment environment (`environment`):

```yaml
environment: production
```

Job dependencies (`needs`):

```yaml
needs: [test, build]
```

Conditional execution (`if`):

```yaml
if: github.ref == 'refs/heads/main'
```

Step Configuration:

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v3

  - name: Setup Node.js
    uses: actions/setup-node@v3
    with:
      node-version: '18'
      cache: 'npm'

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test
    env:
      API_KEY: ${{ secrets.API_KEY }}

  - name: Upload artifacts
    uses: actions/upload-artifact@v3
    if: always()
    with:
      name: test-results
      path: coverage/
```

**Step Types:**

- `uses:` - Run an action from marketplace
- `run:` - Execute shell command
- `name:` - Step description
- `with:` - Action inputs
- `env:` - Environment variables
- `if:` - Conditional execution
- `working-directory:` - Working directory

## 5. Matrix Builds & Parallelization

Matrix Strategy:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [14, 16, 18, 20]
    exclude:
      - os: macos-latest
        node-version: 14
  max-parallel: 6
jobs:
  test:
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
```

**Matrix Options:**

| Option | Purpose | Example |
|---|---|---|
| `matrix` | Define dimensions | `matrix: { os: [ubuntu, windows], version: [1, 2, 3] }` |
| `include` | Add combinations | `include: [{os: ubuntu, version: 4}]` |
| `exclude` | Remove combinations | `exclude: [{os: windows, version: 13}]` |
| `max-parallel` | Concurrent jobs | `max-parallel: 4` |

Real-World Example:

```yaml
strategy:
  matrix:
    node: [14, 16, 18]
    python: [3.8, 3.9, '3.10']
  fail-fast: false
steps:
  - uses: actions/setup-node@v3
    with:
      node-version: ${{ matrix.node }}

  - uses: actions/setup-python@v4
    with:
      python-version: ${{ matrix.python }}
```

## 6. Secrets & Environment Variables

Using Secrets:

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Login to registry
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login -u $ --password-stdin

      - name: Use API key
        run: curl -H "Authorization: Bearer ${{ secrets.API_KEY }}" https://api.example.com

      - name: Database connection
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          DB_USERNAME: ${{ secrets.DB_USERNAME }}
        run: npm test
```

**Secret Types:**

- `GITHUB_TOKEN` - Auto-generated token
- Repository secrets - Private per repo
- Organization secrets - Shared across repos
- Environment secrets - Per environment

**Environment Variables Best Practices:**

DO:

- Use secrets for sensitive data
- Create separate secrets for each service
- Rotate secrets regularly
- Use GitHub's built-in secret masking

DON'T:

- Commit secrets to repository
- Log or echo secrets
- Share secrets in pull requests
- Use plaintext passwords in workflows

**Environment Variables Reference:**

| Variable | Description |
|---|---|
| `github.event_name` | Type of event that triggered workflow |
| `github.ref` | Branch or tag reference |
| `github.sha` | Commit SHA |
| `github.actor` | User who triggered workflow |
| `github.repository` | Repository name (owner/repo) |
| `github.workspace` | Working directory |
| `runner.os` | Runner operating system |

## 7. Using & Creating Actions

**Popular Actions:**

| Action | Purpose |
|---|---|
| `actions/checkout@v3` | Clone repository code |
| `actions/setup-node@v3` | Install Node.js runtime |
| `actions/setup-python@v4` | Install Python runtime |
| `actions/upload-artifact@v3` | Store build outputs |
| `actions/download-artifact@v3` | Retrieve stored artifacts |
| `actions/create-release@v1` | Create GitHub release |

**Creating Custom Actions:**

JavaScript Action:

```yaml
# action.yml
name: 'My Action'
description: 'Does something awesome'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time:
    description: 'The time we greeted'
runs:
  using: 'node16'
  main: 'index.js'
```

Docker Action:

```yaml
# action.yml
name: 'Docker Action'
description: 'Run in Docker container'
runs:
  using: 'docker'
  image: 'docker://ubuntu:latest'
  args:
    - '-c'
    - 'echo "Hello World"'
```

Composite Action:

```yaml
# action.yml
name: 'Composite Action'
description: 'Combination of multiple steps'
inputs:
  version:
    description: 'Version number'
runs:
  using: 'composite'
  steps:
    - run: echo "Version: ${{ inputs.version }}"
      shell: bash
```

## 8. Artifacts & Caching

Upload Artifacts:

```yaml
- name: Run tests
  run: npm test

- name: Upload coverage reports
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: coverage-report
    path: coverage/
    retention-days: 30
```

Download Artifacts:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Download test results
        uses: actions/download-artifact@v3
        with:
          name: coverage-report
```

Caching Dependencies:

```yaml
- name: Cache npm packages
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-

- name: Install dependencies
  run: npm ci
```

**Cache Strategies:**

| Tool | Path | Key |
|---|---|---|
| npm | `~/.npm` | `hashFiles('package-lock.json')` |
| Yarn | `~/.yarn/cache` | `hashFiles('yarn.lock')` |
| pip | `~/.cache/pip` | `hashFiles('requirements.txt')` |
| Docker | `image:tag` | `type=gha` |

## 9. Expressions & Contexts

Expression Syntax:

```yaml
# Boolean logic
if: github.ref == 'refs/heads/main' && github.event_name == 'push'

# Contains function
if: contains(github.event.head_commit.message, '[deploy]')

# startsWith and endsWith
if: startsWith(github.ref, 'refs/tags/v')

# Always, success, failure, cancelled
if: always()
if: success()
if: failure()

# Matrix context
if: matrix.os == 'ubuntu-latest'
```

**Available Contexts:**

| Context | Information | Example |
|---|---|---|
| `github` | Workflow info | `github.repository` |
| `env` | Environment vars | `env.REGISTRY` |
| `secrets` | Secret values | `secrets.API_KEY` |
| `matrix` | Matrix vars | `matrix.node-version` |
| `runner` | Runner info | `runner.os` |
| `job` | Job status | `job.status` |
| `steps` | Step outputs | `steps.setup.outputs.version` |

**Expression Functions:**

- `contains(search, item)` - String contains
- `startsWith(searchString, searchValue)` - Starts with
- `endsWith(searchString, searchValue)` - Ends with
- `format(string, replacements)` - String formatting
- `join(array, separator)` - Join array
- `toJSON(value)` - Convert to JSON
- `fromJSON(value)` - Parse JSON
- `hashFiles(path)` - Hash files

## 10. Conditional Execution & Control Flow

Step Conditions:

```yaml
steps:
  - name: Run on main branch
    if: github.ref == 'refs/heads/main'
    run: echo "On main branch"

  - name: Run on pull request
    if: github.event_name == 'pull_request'
    run: echo "On pull request"

  - name: Run if previous step succeeded
    if: success()
    run: echo "Previous step succeeded"

  - name: Always run
    if: always()
    run: echo "This always runs"

  - name: Run on failure
    if: failure()
    run: echo "Previous step failed"
```

Job Conditions:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/')
    steps:
      - run: echo "Deploying release"

  test:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - run: npm test
```

Complex Conditions:

```yaml
# AND condition
if: github.event_name == 'push' && github.ref == 'refs/heads/main'

# OR condition
if: github.event_name == 'push' || github.event_name == 'pull_request'

# NOT condition
if: '!contains(github.event.head_commit.message, "[skip ci]")'

# Combination
if: |
  (github.event_name == 'push' && github.ref == 'refs/heads/main') ||
  (github.event_name == 'pull_request' && startsWith(github.ref, 'refs/heads/release/'))
```

## 11. Runners & Environments

**GitHub-Hosted Runners:**

| OS | Label | Specs |
|---|---|---|
| Linux | `ubuntu-latest` | 2-core CPU, 7GB RAM, 14GB SSD |
| Windows | `windows-latest` | 2-core CPU, 7GB RAM, 14GB SSD |
| macOS | `macos-latest` | 3-core CPU, 14GB RAM, 14GB SSD |

Self-Hosted Runners:

```bash
# On your machine
./run.sh
```

```yaml
# In workflow
jobs:
  deploy:
    runs-on: [self-hosted, linux, x64]
    steps:
      - run: ./deploy.sh
```

Environments:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://prod.example.com
    steps:
      - name: Deploy to production
        run: npm run deploy
```

**Environment Protection Rules:**

- Required reviewers - Manual approval required
- Deployment branches - Only specific branches can deploy
- Custom deployment protection rules - Advanced controls

## 12. Deployment & CI/CD Patterns

Basic CI/CD Workflow:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm test
      - run: npm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: build
      - name: Deploy
        run: |
          curl -X POST https://api.example.com/deploy -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}" -d '{"version":"${{ github.sha }}"}'
```

Blue-Green Deployment:

```yaml
deploy:
  runs-on: ubuntu-latest
  steps:
    - name: Deploy to green
      run: ./deploy-green.sh
    - name: Health check
      run: curl -f https://green.example.com/health || exit 1
    - name: Switch traffic
      run: |
        aws elbv2 modify-target-group-attributes --target-group-arn \
        arn:aws:elasticloadbalancing:... --attributes \
        Key=deregistration_delay.timeout_seconds,Value=0
```

Canary Deployment:

```yaml
deploy-canary:
  runs-on: ubuntu-latest
  steps:
    - name: Deploy to 5% of users
      run: |
        kubectl set image deployment/app app=myregistry/app:${{ github.sha }} \
        --record --rollout=gradual
    - name: Monitor metrics
      run: ./check-metrics.sh 5

deploy-full:
  needs: deploy-canary
  runs-on: ubuntu-latest
  steps:
    - name: Full rollout
      run: kubectl rollout resume deployment/app
```

## 13. Testing & Code Quality

Running Tests:

```yaml
test:
  runs-on: ubuntu-latest
  services:
    postgres:
      image: postgres:13
      env:
        POSTGRES_PASSWORD: postgres
  steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
    - run: npm install
    - run: npm run test:unit
    - run: npm run test:integration
    - run: npm run test:e2e
```

Code Coverage:

```yaml
- name: Generate coverage
  run: npm run test:coverage

- name: Upload to Codecov
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage/coverage-final.json
    flags: unittests
    fail_ci_if_error: true
```

Linting & Formatting:

```yaml
- name: ESLint
  run: npm run lint

- name: Prettier
  run: npm run format

- name: Type checking
  run: npm run typecheck

- name: Security audit
  run: npm audit --audit-level=moderate
```

## 14. Security & Best Practices

**Security Checklist:**

Security DO's:

- Use secrets for sensitive data
- Limit job permissions
- Review action versions
- Use pinned action versions
- Scan dependencies
- Sign commits and tags
- Use environments for sensitive deployments

Security DON'Ts:

- Store secrets in code
- Log secrets or credentials
- Use broad permission grants
- Trust unverified actions
- Commit `.env` files
- Use `latest` tag for actions

Permissions:

```yaml
permissions:
  contents: read
  pull-requests: write
  deployments: write

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      checks: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v3
```

Scanning for Vulnerabilities:

```yaml
- name: Dependency Check
  run: npm audit

- name: Snyk Scan
  uses: snyk/actions/npm@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

- name: GitHub Security Scanning
  uses: github/super-linter@v4
  env:
    DEFAULT_BRANCH: main
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 15. Debugging & Troubleshooting

Enable Debug Logging:

```yaml
# Set in workflow
env:
  ACTIONS_STEP_DEBUG: true

# Or set in repository secrets
# ACTIONS_STEP_DEBUG: true
```

**Common Issues & Solutions:**

Issue: Workflow not triggering. Check event configuration, branch filters, and path filters:

```yaml
on:
  push:
    branches: [main] # Check branch name
    paths: [src/**] # Check path filters
```

Issue: Secrets not available. Verify secret names and repository access:

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }} # Case-sensitive!
```

Issue: Timeouts. Increase timeout or optimize slow steps:

```yaml
timeout-minutes: 60
```

Issue: Cache not working. Check cache key and restore keys:

```yaml
- uses: actions/cache@v3
  with:
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-
```

**Useful Debugging Commands:**

Dump context:

```yaml
- name: Dump context
  run: |
    echo "Event: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
    echo "Sha: ${{ github.sha }}"
    echo "Actor: ${{ github.actor }}"
```

Check Environment:

```yaml
- name: Check environment
  run: |
    echo "OS: $(uname -a)"
    echo "Node: $(node --version)"
    echo "npm: $(npm --version)"
    env | sort
```

## 16. Quick Reference - Essential Commands

**GitHub CLI Commands for Actions:**

List workflows:

```bash
gh workflow list
gh workflow list --all
```

Trigger workflow:

```bash
gh workflow run deploy.yml -f env=prod
```

List workflow runs:

```bash
gh run list
gh run list --workflow=main.yml
```

View run details:

```bash
gh run view 12345
```

Download artifacts:

```bash
gh run download 12345
```

List secrets:

```bash
gh secret list
```

Create secret:

```bash
gh secret set API_KEY -b "value"
```

Delete secret:

```bash
gh secret remove API_KEY
```

**Common YAML Syntax:**

| Purpose | Syntax | Example |
|---|---|---|
| Define name | `name:` | `name: Deploy` |
| Trigger events | `on:` | `on: [push, pull_request]` |
| Action inputs | `with:` | `with: {node-version: 18}` |
| Environment variables | `env:` | `env: {NODE_ENV: production}` |
| Condition | `if:` | `if: success()` |
| Execute command | `run:` | `run: npm test` |
| Use action | `uses:` | `uses: actions/checkout@v3` |

## 17. Best Practices & Pro Tips

**Workflow Best Practices:**

- Keep workflows DRY: Use reusable workflows to avoid duplication
- Use specific action versions: Pin to exact versions, not latest
- Implement caching: Cache dependencies for faster builds
- Fail fast: Run quick checks before expensive operations
- Use concurrency: Control parallel execution with concurrency limits
- Document everything: Comment workflows for maintainability
- Test locally: Use the `act` tool to test workflows locally
- Monitor costs: Track usage to avoid surprises
- Use proper permissions: Apply least privilege principle
- Implement approval gates: Use environments for critical deployments

**Performance Optimization:**

- Run independent jobs simultaneously
- Test multiple versions in parallel
- Skip unnecessary steps with conditions
- Run quick tests before slow ones
- Compress and clean up artifacts

**Common Gotchas to Avoid:**

- Using `latest` tag for actions (use specific version)
- Storing secrets in code or logs
- Not testing workflows locally
- Ignoring job dependencies
- Over-using wildcard permissions
- Not setting timeouts
- Forgetting to clean up artifacts
- Hardcoding sensitive configuration

## 18. Cost Management & Optimization

**GitHub Actions Pricing:**

- Public repositories: Free unlimited minutes
- Private repositories: 2,000 minutes/month free, then $0.25/1000 minutes
- Windows runners: 2x minute multiplier
- macOS runners: 10x minute multiplier

Cost Optimization Techniques:

```yaml
# 1. Use concurrency to cancel old runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# 2. Limit matrix combinations
strategy:
  matrix:
    node: [16, 18] # Test fewer versions
  max-parallel: 2 # Limit concurrent jobs

# 3. Cache aggressively
- uses: actions/cache@v3
  with:
    key: ${{ hashFiles('**/lock') }}

# 4. Use conditional execution
if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/')

# 5. Minimize artifact retention
- uses: actions/upload-artifact@v3
  with:
    retention-days: 3 # Delete old artifacts

# 6. Use self-hosted runners for heavy work
runs-on: [self-hosted, linux]
```

**Monitoring Costs:**

Set up GitHub Actions billing alerts and review monthly usage in:

Settings -> Billing and plans -> Usage overview

---

*Source: adapted from the Github Actions cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
