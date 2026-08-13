# Nexus Repository Cheatsheet

Quick reference for Nexus Repository Manager 3.x — installation, configuration, repository formats, REST API, security, backup, and troubleshooting.

## 🛠 Installation & Setup

Install on Linux

```bash
cd /opt

wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvzf latest-unix.tar.gz

ln -s nexus-3.* nexus

useradd -r -s /bin/false nexus

chown -R nexus:nexus /opt/nexus /opt/sonatype-work
```

Configure as Service

```bash
vi /etc/systemd/system/nexus.service
```

```ini
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
User=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
systemctl enable nexus
systemctl start nexus
```

Default Admin Password

```bash
cat /opt/sonatype-work/nexus3/admin.password
```

> Important: Change the default password immediately after first login!

## 📁 Key Paths & URLs

| Item | Path / URL |
|---|---|
| Web UI | `http://localhost:8081` |
| Install Dir | `/opt/nexus` |
| Data Dir | `/opt/sonatype-work/nexus3` |
| Config | `/opt/sonatype-work/nexus3/etc` |
| Logs | `/opt/sonatype-work/nexus3/log` |
| Blob Store | `/opt/sonatype-work/nexus3/blobs` |
| JVM Config | `/opt/nexus/bin/nexus.vmoptions` |

Important Log Files

| File | Purpose |
|---|---|
| `nexus.log` | Application logs |
| `request.log` | HTTP requests |
| `audit.log` | Security events |
| `tasks/*.log` | Scheduled tasks |

## ⚙ Service Management

Basic Commands

```bash
systemctl start nexus
systemctl stop nexus
systemctl restart nexus
systemctl status nexus

# View logs in real time (journalctl)
journalctl -u nexus -f

# Or tail the log file directly
tail -f /opt/sonatype-work/nexus3/log/nexus.log
```

JVM Memory Configuration

```bash
# Edit nexus.vmoptions
vi /opt/nexus/bin/nexus.vmoptions
```

```
-Xms2G
-Xmx2G
-XX:MaxDirectMemorySize=2G
-XX:+UnlockDiagnosticVMOptions
-XX:+LogVMOutput
-XX:LogFile=/opt/sonatype-work/nexus3/log/jvm.log
```

> Tip: Set `-Xms` equal to `-Xmx` to prevent heap resizing.

## 📦 Maven Configuration

settings.xml (`~/.m2/settings.xml`)

```xml
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus.company.com:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>{encrypted-password}</password>
    </server>
  </servers>
</settings>
```

pom.xml Configuration

```xml
<distributionManagement>
  <repository>
    <id>nexus</id>
    <url>http://nexus.company.com:8081/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>nexus</id>
    <url>http://nexus.company.com:8081/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```

Deploy Artifact

```bash
mvn clean deploy
```

## 🐘 Gradle Configuration

build.gradle

```groovy
repositories {
    maven {
        url "http://nexus.company.com:8081/repository/maven-public/"
        credentials {
            username "admin"
            password "password123"
        }
    }
}

publishing {
    publications {
        maven(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            url = version.endsWith('SNAPSHOT') ?
                "http://nexus.company.com:8081/repository/maven-snapshots/" :
                "http://nexus.company.com:8081/repository/maven-releases/"
            credentials {
                username "admin"
                password "password123"
            }
        }
    }
}
```

gradle.properties

```properties
nexusUrl=http://nexus.company.com:8081
nexusUsername=admin
nexusPassword=password123
```

## 🐳 Docker Registry

Create Docker Repository

```
Type: docker (hosted)
HTTP Port: 5000
Enable Docker V1 API: No (deprecated)
Blob Store: default
```

Docker Login

```bash
docker login nexus.company.com:5000
# Username: admin
# Password: ********
```

Tag and Push Image

```bash
docker tag myapp:latest nexus.company.com:5000/myapp:latest
docker push nexus.company.com:5000/myapp:latest
```

Pull Image

```bash
docker pull nexus.company.com:5000/myapp:latest
```

Configure Docker Daemon

```bash
# /etc/docker/daemon.json
{
  "insecure-registries": ["nexus.company.com:5000"]
}
```

> Production: Use HTTPS with valid certificates, not insecure registries!

## 📦 npm Configuration

Set Registry

```bash
npm config set registry http://nexus.company.com:8081/repository/npm-group/

# Set authentication
npm config set //nexus.company.com:8081/repository/npm-group/:_auth $(echo -n 'admin:password123' | base64)
```

.npmrc File

```ini
registry=http://nexus.company.com:8081/repository/npm-group/
//nexus.company.com:8081/repository/npm-group/:_auth=YWRtaW46cGFzc3dvcmQxMjM=
always-auth=true
```

Publish Package

```bash
npm publish --registry http://nexus.company.com:8081/repository/npm-hosted
```

Install from Nexus

```bash
npm install package-name
```

## 🐍 Python/pip Configuration

pip.conf (`~/.pip/pip.conf`)

```ini
[global]
index-url = http://nexus.company.com:8081/repository/pypi-group/simple
trusted-host = nexus.company.com
```

Install Package

```bash
pip install package-name
```

Upload with twine

```bash
# .pypirc
[distutils]
index-servers = nexus

[nexus]
repository: http://nexus.company.com:8081/repository/pypi-hosted/
username: admin
password: password123
```

```bash
# Upload
twine upload --repository nexus dist/*
```

## 🔗 REST API Examples

Authentication

```bash
curl -u admin:password123 \
  http://nexus.company.com:8081/service/rest/v1/status
```

List Repositories

```bash
curl -u admin:password123 \
  http://nexus.company.com:8081/service/rest/v1/repositories
```

Search Components

```bash
curl -u admin:password123 \
  "http://nexus.company.com:8081/service/rest/v1/search?name=myapp&format=maven2"
```

Upload Asset

```bash
curl -u admin:password123 \
  -F "maven2.groupId=com.example" \
  -F "maven2.artifactId=myapp" \
  -F "maven2.version=1.0.0" \
  -F "maven2.asset1=@myapp-1.0.0.jar" \
  -F "maven2.asset1.extension=jar" \
  http://nexus.company.com:8081/service/rest/v1/components?repository=maven-releases
```

Delete Component

```bash
curl -X DELETE -u admin:password123 \
  http://nexus.company.com:8081/service/rest/v1/components/{component-id}
```

## 📚 Repository Types

| Type | Purpose |
|---|---|
| Hosted | Store your artifacts — internal libraries, releases |
| Proxy | Cache remote repos — Maven Central, npm registry |
| Group | Combine multiple repos — single URL for builds |

Supported Formats

- Maven (Java)
- npm (JavaScript)
- Docker (Containers)
- PyPI (Python)
- NuGet (.NET)
- RubyGems (Ruby)
- Helm (Kubernetes)
- APT/YUM (Linux packages)
- Raw (any file type)

## 🧹 Cleanup Policies

Create Cleanup Policy

- Navigate: System → Repository → Cleanup Policies
- Click: Create Cleanup Policy
- Name: `remove-old-snapshots`
- Format: Maven2
- Criteria: Last downloaded < 30 days

Common Cleanup Criteria

| Criteria | Example |
|---|---|
| Last downloaded | > 30 days |
| Last updated | > 90 days |
| Asset name matches | `.*-SNAPSHOT.*` |

Run Cleanup Task

- Navigate: System → Tasks
- Create: Admin - Compact blob store
- Schedule: Daily at 2:00 AM

## 🔐 Security & Access Control

Common Roles

| Role | Access |
|---|---|
| `nx-admin` | Full system access |
| `nx-deployer` | Deploy to hosted repos |
| `nx-developer` | Download from all repos |
| `nx-anonymous` | Public repo access only |

Create User

- Navigate: Security → Users
- Click: Create local user
- Assign appropriate roles

Enable LDAP

- Navigate: System → Security → LDAP
- Configure connection details
- Map LDAP groups to Nexus roles
- Test authentication

API Token (Recommended)

- Navigate: User menu → User Settings → User Token
- Click "Access User Token"
- Use the token instead of a password for authentication

## 💾 Backup & Restore

What to Backup

- Blob stores: `/opt/sonatype-work/nexus3/blobs/`
- Database: `/opt/sonatype-work/nexus3/db/`
- Configuration: `/opt/sonatype-work/nexus3/etc/`
- Keystores: SSL certificates

Backup Script

```bash
#!/bin/bash

BACKUP_DIR="/backup/nexus/$(date +%Y%m%d)"
NEXUS_DATA="/opt/sonatype-work/nexus3"

mkdir -p $BACKUP_DIR

systemctl stop nexus

# Backup blob store
tar -czf $BACKUP_DIR/blobs.tar.gz $NEXUS_DATA/blobs/

# Backup database
tar -czf $BACKUP_DIR/db.tar.gz $NEXUS_DATA/db/

# Backup config
tar -czf $BACKUP_DIR/etc.tar.gz $NEXUS_DATA/etc/

systemctl start nexus
```

Restore

```bash
systemctl stop nexus

tar -xzf blobs.tar.gz -C /
tar -xzf db.tar.gz -C /
tar -xzf etc.tar.gz -C /

chown -R nexus:nexus /opt/sonatype-work
systemctl start nexus
```

## 🔧 Troubleshooting

Check Service Status

```bash
systemctl status nexus
journalctl -u nexus -n 150
```

View Logs

```bash
tail -f /opt/sonatype-work/nexus3/log/nexus.log | grep ERROR
```

Check Disk Space

```bash
df -h /opt/sonatype-work
```

Check Memory Usage

```bash
ps -o rss,cmd -C java | grep nexus
free -h
```

Common Issues

| Issue | Fix |
|---|---|
| Won't start | Check logs, verify Java version, disk space |
| Slow performance | Increase JVM heap, check disk I/O |
| Out of memory | Edit nexus.vmoptions, increase `-Xmx` |
| 404 on artifacts | Check repository, permissions, proxy config |

Port Conflicts

```bash
# Check if port 8081 is in use
netstat -tlnp | grep 8081

# Change the port in nexus-default.properties
vi /opt/nexus/etc/nexus-default.properties
# application-port=8082
```

## 🚀 Performance Tuning

JVM Tuning

```
# Recommended for 8GB RAM server
-Xms4G
-Xmx4G
-XX:MaxDirectMemorySize=4G
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

PostgreSQL (Production)

```bash
# Install PostgreSQL 10+

# Create database and user
createdb nexus
createuser nexus_user
```

```properties
# Configure Nexus to use PostgreSQL
# nexus.properties
nexus.datastore.enabled=true
nexus.datastore.nexus.jdbcUrl=jdbc:postgresql://localhost:5432/nexus
nexus.datastore.nexus.username=nexus_user
nexus.datastore.nexus.password=secret
```

Storage Optimization

- Use SSD for blob stores
- Enable blob store deduplication
- Schedule cleanup tasks nightly
- Archive old releases to S3

Caching

- Configure proxy repositories with longer TTL
- Increase component cache size
- Use CDN for geographically distributed teams

## 🔗 Useful URLs & Resources

Nexus URLs

| Purpose | URL |
|---|---|
| Web UI | `http://localhost:8081` |
| System Info | Admin → Support → System Information |
| Metrics | Admin → Support → Metrics |
| API Docs | `http://localhost:8081/swagger-ui/` |
| Health Check | `http://localhost:8081/service/rest/v1/status` |

Documentation

- Official Docs: https://help.sonatype.com/repomanager3
- REST API: https://help.sonatype.com/repomanager3/rest-and-integration-api
- Downloads: https://www.sonatype.com/download-oss-sonatype

## 🔄 Jenkins Integration

Global Tool Configuration

```xml
<!-- Maven settings.xml in Jenkins -->
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASSWORD}</password>
    </server>
  </servers>
</settings>
```

Pipeline Script

```groovy
pipeline {
    agent any
    environment {
        NEXUS_URL = 'http://nexus.company.com:8081'
        NEXUS_REPO = 'maven-releases'
        NEXUS_CREDS = credentials('nexus-credentials')
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy -DaltDeploymentRepository=nexus::default::${NEXUS_URL}/repository/${NEXUS_REPO}'
            }
        }
    }
}
```

## 📋 Quick Command Reference

| Task | Command |
|---|---|
| Start Nexus | `systemctl start nexus` |
| Stop Nexus | `systemctl stop nexus` |
| View logs | `tail -f /opt/sonatype-work/nexus3/log/nexus.log` |
| Check status | `systemctl status nexus` |
| Admin password | `cat /opt/sonatype-work/nexus3/admin.password` |
| Disk usage | `du -sh /opt/sonatype-work/nexus3/blobs` |
| Backup | `tar -czf nexus-backup.tar.gz /opt/sonatype-work` |
| Test connection | `curl http://localhost:8081` |

---

*Source: adapted from the Nexus Repository cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
