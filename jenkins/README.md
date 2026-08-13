# Jenkins Cheatsheet

Complete, practical reference guide for Jenkins CI/CD automation and pipeline development.

## 🎯 Installation & Setup

### System Requirements

**Requirements:**
- OS: Linux, Windows, macOS
- Memory: 512MB min (1GB+ for production)
- Disk: 1GB+ for production
- Java: OpenJDK 11+
- Browser: Chrome, Firefox, Safari, Edge

**Docker Setup:**
```bash
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
docker exec <container_id> cat /var/jenkins_home/secrets/initialAdminPassword
```

**Ubuntu Install:**
```bash
sudo apt-get update
sudo apt-get install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y jenkins
sudo systemctl start jenkins
```

### Configuration

**Initial Setup:**
1. Access `http://localhost:8080`
2. Copy admin password from `/var/jenkins_home/secrets/initialAdminPassword`
3. Install suggested plugins
4. Create first admin user
5. Save and finish

**System Configuration:**

`Manage Jenkins > System Configuration`
- Configure system
- Set Home Directory
- Configure Shell
- Setup Email

## 📝 Declarative Pipeline

### Basic Structure

**Pipeline Anatomy:**
```groovy
pipeline {
  agent any

  options {
    timestamps()
    timeout(time: 1, unit: 'HOURS')
  }

  parameters {
    string(name: 'VERSION', defaultValue: '1.0')
  }

  triggers {
    cron('H/15 * * * *')
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building...'
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}
```

**Minimal Pipeline:**
```groovy
pipeline {
  agent any
  stages {
    stage('Hello') {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

### Parameters

**Parameter Types:**
- `string`: Text input
- `choice`: Dropdown selection
- `booleanParam`: True/False
- `password`: Hidden input
- `run`: Choose from builds

**Parameter Example:**
```groovy
parameters {
  string(name: 'NAME', defaultValue: 'World')
  choice(name: 'ENV', choices: ['dev', 'staging', 'prod'])
  booleanParam(name: 'DEBUG', defaultValue: false)
}

steps {
  echo "Hello ${params.NAME}"
  echo "Environment: ${params.ENV}"
}
```

## 🔀 Stages & Steps

### Parallel Execution

**Parallel Stages:**
```groovy
stages {
  stage('Test') {
    parallel {
      stage('Unit Tests') {
        steps { sh 'mvn test' }
      }
      stage('Integration') {
        steps { sh 'mvn verify' }
      }
      stage('Lint') {
        steps { sh 'npm run lint' }
      }
    }
  }
}
```

**Sequential Steps:**
```groovy
steps {
  echo 'Step 1'
  sh 'npm install'
  sh 'npm run build'
  sh 'npm test'
}
```

### Conditional Execution

**When Conditions:**
```groovy
stage('Deploy') {
  when {
    branch 'main'
    environment name: 'ENV', value: 'production'
    expression { currentBuild.result == null }
  }
  steps { sh './deploy.sh' }
}
```

**When Operators:**
- `always`: Always run
- `branch`: Specific branch
- `environment`: Environment variable
- `expression`: Groovy expression
- `changeset`: Files changed
- `triggeredBy`: Build trigger

## 🔐 Credentials & Environment

### Using Credentials

**String Credentials:**
```groovy
pipeline {
  environment {
    API_KEY = credentials('api-key-id')
  }
  stages {
    stage('Test') {
      steps {
        sh 'echo $API_KEY'
      }
    }
  }
}
```

**SSH Credentials:**
```groovy
sshagent(['ssh-key-credentials-id']) {
  sh 'ssh user@host "command"'
  sh 'git clone git@github.com:user/repo.git'
}
```

**Docker Credentials:**
```groovy
environment {
  DOCKER_CREDS = credentials('docker-hub-creds')
}
steps {
  sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
}
```

### Environment Variables

**Built-in Variables:**
- `BUILD_NUMBER`: Build number
- `BUILD_ID`: Build ID
- `JOB_NAME`: Job name
- `WORKSPACE`: Build directory
- `GIT_COMMIT`: Git commit hash
- `BRANCH_NAME`: Git branch

**Custom Variables:**
```groovy
pipeline {
  environment {
    APP_NAME = 'MyApp'
    VERSION = '1.0.0'
    BUILD_TIME = sh(script: "date +%Y%m%d_%H%M%S", returnStdout: true)
  }
}
```

## 🔌 Plugins & Integrations

### Popular Plugins

**Pipeline Plugins:**
- Pipeline: Suite of plugins
- Blue Ocean: UI improvement
- GitHub: GitHub integration
- Docker: Container support
- Kubernetes: K8s support

**Notification Plugins:**
- Email Extension: Advanced email
- Slack: Slack notifications
- MS Teams: Teams integration
- PagerDuty: Incident management
- Datadog: Metrics collection

### Plugin Installation

**Install Plugin:**

`Manage Jenkins > Plugin Manager`
- Available: Search for plugin
- Install Without Restart
- Manage Plugins > Updates

**Docker Plugin:**
```groovy
pipeline {
  agent {
    docker {
      image 'maven:3.8'
      args '-v /root/.m2:/root/.m2'
    }
  }
  stages {
    stage('Build') {
      steps { sh 'mvn clean package' }
    }
  }
}
```

## 📤 Triggers

### Build Triggers

**SCM Polling:**
```groovy
pollSCM('H/15 * * * *')  // Every 15 min
pollSCM('H 2 * * *')     // Daily at 2am
pollSCM('0 0 * * 0')     // Weekly Sunday
```

**GitHub Webhook:**
```groovy
githubPush()  // Triggered on push
```
GitHub webhook URL: `http://jenkins/github-webhook/`

Configure under: `Repository Settings > Webhooks`

**Cron Schedule:**
```groovy
cron('0 9 * * 1-5')      // 9am Mon-Fri
cron('0 0 * * *')        // Daily midnight
cron('H H * * *')        // Daily (hash)
```

### Trigger Events

**Upstream Trigger:**
```groovy
upstream(upstreamProjects: 'other-job',
         threshold: hudson.model.Result.SUCCESS)
```

**Timer Trigger:**
```yaml
# In job config
triggers:
  - type: cron
    schedule: 'H * * * *'
```

## ✅ Best Practices & Patterns

### Pipeline Best Practices

**DO's:**
- ✓ Use Declarative Pipeline
- ✓ Store Jenkinsfile in SCM
- ✓ Use shared libraries
- ✓ Implement error handling
- ✓ Use timeouts
- ✓ Log appropriately

**DON'Ts:**
- ✗ Hardcode credentials
- ✗ Use master for builds
- ✗ Skip testing
- ✗ Ignore security
- ✗ Write complex Groovy code
- ✗ Perform manual deployments

### Error Handling & Recovery

**Post Actions:**
```groovy
post {
  always {
    cleanWs()
    junit 'target/**/*.xml'
  }
  success {
    echo 'Build successful'
  }
  failure {
    emailext(subject: 'Build failed')
  }
  unstable {
    echo 'Build unstable'
  }
  aborted {
    echo 'Build was aborted'
  }
}
```

**Try-Catch:**
```groovy
try {
  sh 'npm test'
} catch (exc) {
  echo 'Test failed!'
  currentBuild.result = 'UNSTABLE'
}
```

## 🐛 Troubleshooting & Debugging

### Common Issues

**Build Fails:**
1. Check logs in job output
2. Verify agent availability
3. Check plugin versions
4. Test stage independently
5. Review recent changes

**Pipeline Syntax:**

Use the declarative linter:

`Manage Jenkins > Declarative Linter`

Or validate from the command line:
```bash
curl -X POST -F "jenkinsfile=<Jenkinsfile" http://localhost:8080/pipeline-model-converter/validate --user user:apitoken
```

### Debugging Techniques

**Debug Output:**
```bash
sh 'set -x'        # Enable bash debug
echo "Variable: ${VAR}"
sh 'env | sort'     # Print all vars
sh 'pwd && ls -la'
```

**Jenkins CLI Debugging:**
```bash
java -jar jenkins-cli.jar -s http://localhost:8080 get-job job-name
java -jar jenkins-cli.jar -s http://localhost:8080 console job-name build-number
```

## 📋 Jenkins Files & Directories

### Important Locations

**Key Paths:**
```bash
JENKINS_HOME=/var/lib/jenkins
# Jobs:    $JENKINS_HOME/jobs/
# Plugins: $JENKINS_HOME/plugins/
# Config:  $JENKINS_HOME/config.xml
# Logs:    $JENKINS_HOME/logs/
```

**Configuration Files:**
```
jenkins.plugins.shiningpanda.ShiningPandaGlobalConfig.xml
org.jenkinsci.plugins.github_branch_source.GitHubSCMSource.xml
org.jenkinsci.plugins.workflow.flow.FlowDefinition.xml
```

## 📊 Jenkins Concepts Reference

### Key Terms

**Pipeline Elements:**
- `Agent`: Where jobs run
- `Stage`: Logical division
- `Step`: Individual task
- `Post`: Final actions
- `Trigger`: Start condition

**Job Types:**
- `Freestyle`: Traditional jobs
- `Pipeline`: Code-as-config
- `Multibranch`: Multiple branches
- `External`: Non-Jenkins projects

## 📋 Quick Reference Commands

| Command/Section | Purpose | Example/Usage |
|---|---|---|
| `pipeline` | Define pipeline | `pipeline { ... }` |
| `agent` | Execution location | `agent any` |
| `stages` | Job phases | `stages { stage(...) }` |
| `steps` | Tasks in stage | `steps { sh '...' }` |
| `post` | Cleanup/notification | `post { always {...} }` |
| `when` | Stage conditions | `when { branch 'main' }` |
| `parallel` | Concurrent stages | `parallel { stage(...) }` |
| `parameters` | Input parameters | `parameters { string(...) }` |
| `triggers` | Start conditions | `triggers { cron(...) }` |
| `environment` | Global variables | `environment { VAR = '...' }` |
| `options` | Pipeline options | `options { timeout(...) }` |
| `credentials` | Secret management | `credentials('id')` |
| `archiveArtifacts` | Save build output | `archiveArtifacts 'build/**'` |
| `junit` | Test results | `junit 'test-results/*.xml'` |

## 🔌 Popular Plugins Reference

**Pipeline Plugins:**
- Pipeline
- Blue Ocean
- Shared Library
- Stage View

**SCM Plugins:**
- GitHub
- GitLab
- Bitbucket
- Gitea

**Container Plugins:**
- Docker
- Kubernetes
- OpenShift
- Amazon ECS

**Notification:**
- Email
- Slack
- MS Teams
- PagerDuty

**Testing:**
- JUnit
- Cobertura
- SonarQube
- Jacoco

**Build Tools:**
- Maven
- Gradle
- NodeJS
- Ant

## ✅ Jenkins Best Practices Checklist

**✓ Security**
- Use Jenkins credentials manager for all secrets
- Enable CSRF protection
- Use security realms (LDAP, GitHub, etc.)
- Implement audit logging
- Keep Jenkins and plugins updated

**✓ Pipeline Management**
- Store Jenkinsfile in Git repository
- Use shared libraries for reusable code
- Implement proper error handling
- Add meaningful logging
- Set appropriate timeouts

**✓ Agent Management**
- Use labels for agent grouping
- Configure agent pools correctly
- Monitor agent health
- Use dedicated agents for heavy jobs
- Implement auto-scaling where possible

**✓ Maintenance**
- Regular backups of `JENKINS_HOME`
- Monitor disk space
- Clean up old logs and artifacts
- Review plugin compatibility
- Document pipeline logic

**💡 Performance Tips:**
- Enable Distributed Builds
- Use Build Cache
- Implement Parallel Execution
- Clean workspace between builds
- Monitor build performance metrics

**⚠️ Common Pitfalls:**
- Don't use master for builds
- Avoid complex Groovy code
- Don't ignore test failures
- Avoid long-running stages
- Don't skip documentation

---
*Source: adapted from the Jenkins cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
