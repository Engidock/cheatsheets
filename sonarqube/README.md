# SonarQube Cheatsheet

A condensed, look-up reference for SonarQube architecture, scanner properties, server configuration, execution commands, the Web API, and day-to-day troubleshooting.

## 1. Architecture & System Reference

### 1.1 Default Ports

| Port | Component | Access | Notes |
|------|-----------|--------|-------|
| 9000 | Web Server (Tomcat) | Public / Ingress | The main UI and API port. Can be changed via `sonar.web.port`. |
| 9001 | Elasticsearch | Loopback Only | Strictly internal. Never expose this port publicly. |
| 9092 | H2 Database | Internal Only | Only active if using the embedded evaluation database. |
| 5432 | PostgreSQL | Network | The recommended external database connection. |

### 1.2 File System Paths (Linux/Docker)

| Path | Content | Backup Strategy |
|------|---------|------------------|
| `/opt/sonarqube/conf` | `sonar.properties`, `wrapper.conf` | Backup required. |
| `/opt/sonarqube/extensions` | Plugins (`/plugins`) and drivers (`/jdbc-driver`) | Backup required. |
| `/opt/sonarqube/data` | Elasticsearch indices (`/es7`) | Volatile. Do not back up — can be rebuilt from the DB. |
| `/opt/sonarqube/logs` | `sonar.log`, `web.log`, `ce.log`, `es.log` | Rotate and monitor. |

## 2. Scanner Properties (sonar-project.properties)

These keys control the analysis. They can be set in the properties file, or passed via the command line (`-Dkey=value`).

| Property Key | Required? | Description | Example |
|---------------|-----------|--------------|---------|
| `sonar.projectKey` | YES | Unique ID. Cannot contain spaces. | `my-company:payment-api` |
| `sonar.sources` | YES | Paths to source code. | `src/main/java, src/js` |
| `sonar.host.url` | YES | Server address. | `http://localhost:9000` |
| `sonar.token` | YES | Auth token. | `sqp_a1b2c3d4...` |
| `sonar.exclusions` | NO | Files to completely ignore. | `**/*.min.js, **/vendor/**` |
| `sonar.tests` | NO | Path to test files (for metrics). | `src/test/java` |
| `sonar.test.exclusions` | NO | Ignore specific test files. | `**/*IntegrationTest.java` |
| `sonar.coverage.exclusions` | NO | Files analyzed for bugs but ignored for coverage calculation. | `**/config/**, **/dto/**` |
| `sonar.cpd.exclusions` | NO | Files ignored for duplication detection. | `**/generated/**` |
| `sonar.qualitygate.wait` | NO | Block the pipeline until the gate status is returned. | `true` |
| `sonar.branch.name` | DEV+ | Name of the branch being analyzed. | `feature/login` |
| `sonar.pullrequest.key` | DEV+ | PR number for decoration. | `105` |
| `sonar.pullrequest.branch` | DEV+ | Source branch of the PR. | `fix/bug-123` |
| `sonar.pullrequest.base` | DEV+ | Target branch of the PR. | `main` |

> `DEV+` = requires branch/PR analysis features available from Developer Edition and above.

## 3. Server Configuration (sonar.properties)

These settings configure the SonarQube instance. Changes require a restart.

### 3.1 Database Connection

```properties
sonar.jdbc.username=sonar
sonar.jdbc.password=SecurePass!
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonar
```

### 3.2 Web Server (UI) Tuning

```properties
sonar.web.host=0.0.0.0        # Listen on all interfaces
sonar.web.port=9000
sonar.web.context=/sonarqube  # If using a reverse proxy subpath
sonar.web.javaOpts=-Xmx2G -Xms1G -XX:+HeapDumpOnOutOfMemoryError
```

### 3.3 Compute Engine (Worker) Tuning

```properties
sonar.ce.javaOpts=-Xmx4G -Xms2G -XX:+HeapDumpOnOutOfMemoryError
sonar.ce.workerCount=2   # Only effective in Enterprise Edition
```

### 3.4 Elasticsearch Tuning

```properties
sonar.search.javaOpts=-Xmx2G -Xms2G
sonar.es.bootstrap.checks.disable=true   # Only for non-production!
```

## 4. Execution Commands

### 4.1 Maven (Java)

```bash
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=my-app \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=$TOKEN
```

### 4.2 Gradle (Android/Java/Kotlin)

```bash
./gradlew sonar \
  -Dsonar.token=$TOKEN
```

### 4.3 .NET (C# / VB.NET)

```bash
# Step 1: Begin (hooks into MSBuild)
dotnet sonarscanner begin /k:"my-app" /d:sonar.token="..."

# Step 2: Build
dotnet build

# Step 3: End (uploads report)
dotnet sonarscanner end /d:sonar.token="..."
```

### 4.4 NPM (JavaScript / TypeScript)

```bash
# Install dependency
npm install sonarqube-scanner --save-dev

# Run scan
npm run sonar
```

## 5. Metrics & API Reference

### 5.1 Common Metric Keys

Use these JSON keys when querying the API or defining Quality Gates.

| Domain | Keys |
|--------|------|
| Reliability | `bugs`, `new_bugs`, `reliability_rating` |
| Security | `vulnerabilities`, `security_hotspots`, `security_rating` |
| Maintainability | `code_smells`, `sqale_index` (debt in minutes), `sqale_debt_ratio` |
| Coverage | `coverage`, `new_coverage`, `uncovered_lines` |
| Duplication | `duplicated_lines_density`, `duplicated_blocks` |
| Complexity | `cognitive_complexity`, `complexity` |

### 5.2 Essential Web API Endpoints

Auth: Basic Auth using a User Token (no password required).

```bash
# 1. Provision a project
POST api/projects/create?name=MyProject&project=my-key

# 2. Add a group to a permission template
POST api/permissions/add_group_to_template?templateName=Default&groupName=sonar-admins&permission=admin

# 3. Get Quality Gate status
GET api/qualitygates/project_status?projectKey=my-key

# 4. Search users
GET api/users/search?q=alice

# 5. Create a token
POST api/user_tokens/generate?name=CiToken&type=PROJECT_ANALYSIS_TOKEN&projectKey=my-key
```

## 6. Troubleshooting & Maintenance

### 6.1 Database Housekeeping (SQL)

Run these on PostgreSQL if the DB grows too large.

```sql
-- Delete old snapshots > 2 years
DELETE FROM snapshots WHERE created_at < NOW() - INTERVAL '2 years';

-- Vacuum (reclaim space)
VACUUM FULL;
```

### 6.2 Elasticsearch Recovery

If SonarQube fails to start with `"ES is closed"` or `"Lock held"`:

1. Stop SonarQube.
2. Go to `/opt/sonarqube/data`.
3. Delete the `es7` (or `es8`) folder. Do **NOT** delete the DB.
4. Start SonarQube. It will rebuild the index from the DB (this takes time).

### 6.3 Password Reset (Emergency)

If you lost the admin password and LDAP is down:

```sql
-- Hash for "admin" (BCrypt)
UPDATE users SET crypted_password = '$2a$12$...' WHERE login = 'admin';
```

## 7. Regex Reference for Permissions

Common patterns for Permission Templates.

| Goal | Regex Pattern | Matches |
|------|----------------|---------|
| Specific Team | `^finance-.*` | `finance-api`, `finance-web` |
| Public Apps | `.*-public$` | `website-public`, `api-public` |
| Legacy | `legacy-(java\|cpp)-.*` | `legacy-java-app`, `legacy-cpp-tool` |

**Final tip:** Always verify your "New Code" definition (Administration > General > New Code). Setting it to "Previous Version" is the safest bet for release-based development. Setting it to "Reference Branch" is best for feature-branch workflows.

---
*Source: adapted from the SonarQube cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
