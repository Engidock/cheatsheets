# Apache Maven Cheatsheet

Comprehensive, practical quick-reference for Apache Maven build automation and project management.

## 🛠️ Installation & Configuration

**Install Maven:**
```bash
# Linux/Mac
brew install maven

# Windows (with Chocolatey)
choco install maven

# Manual: Download from maven.apache.org and add to PATH
export PATH=$PATH:/path/to/maven/bin
```

**Verify Installation:**
```bash
mvn --version
mvn -v
```

**Maven Configuration Files:**

| File | Location | Purpose |
|---|---|---|
| `settings.xml` (User) | `~/.m2/settings.xml` | User-level configuration |
| `pom.xml` (Project) | Project root | Project configuration |
| `settings-security.xml` | `~/.m2/settings-security.xml` | Encrypted master password |

## 🚀 Basic Maven Commands

**Build Lifecycle Commands:**

| Command | Phase | Description |
|---|---|---|
| `mvn clean` | clean | Remove build directory (`target/`) |
| `mvn compile` | compile | Compile source code |
| `mvn test` | test | Run unit tests |
| `mvn package` | package | Create JAR/WAR |
| `mvn install` | install | Install to local repo (`~/.m2`) |
| `mvn deploy` | deploy | Deploy to remote repo |

**Common Command Combinations:**
```bash
# Full build
mvn clean install

# Skip tests
mvn clean install -DskipTests

# Single module in multi-module
mvn clean install -pl :module-name

# Offline mode
mvn clean install -o

# Parallel builds (threads)
mvn clean install -T 1C

# Debug mode
mvn -X clean install

# Show errors
mvn -e clean install

# Run single test
mvn test -Dtest=TestClassName

# Run specific test method
mvn test -Dtest=TestClassName#methodName
```

**Dependency Commands:**
```bash
# View dependency tree
mvn dependency:tree

# View with verbose
mvn dependency:tree -Dverbose

# Analyze dependencies
mvn dependency:analyze

# Download all dependencies
mvn dependency:resolve

# View effective POM
mvn help:effective-pom

# Show active profiles
mvn help:active-profiles
```

## 📄 POM Configuration Reference

**Minimal POM Structure:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
  <name>My Application</name>
  <description>My App Description</description>
  <url>http://example.com</url>
</project>
```

**Parent POM Template:**
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>parent-pom</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <modules>
    <module>module-a</module>
    <module>module-b</module>
  </modules>
</project>
```

**Child POM Template:**
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.example</groupId>
    <artifactId>parent-pom</artifactId>
    <version>1.0.0</version>
  </parent>
  <artifactId>child-module</artifactId>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## 📦 Dependency Configuration

**Dependency Scopes:**

| Scope | Available at Compile | Available at Runtime | Example |
|---|---|---|---|
| `compile` (default) | Yes | Yes | Spring Framework |
| `test` | No | No (test only) | JUnit, Mockito |
| `runtime` | No | Yes | MySQL Driver |
| `provided` | Yes | No (container provides) | Servlet API |
| `optional` | No | No (optional feature) | Logger |
| `import` (BOM only) | N/A | N/A | Spring BOM |

**Dependency Configuration Example:**
```xml
<dependencies>

  <!-- Compile dependency -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.0</version>
  </dependency>

  <!-- Test dependency -->
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
  </dependency>

  <!-- Runtime dependency -->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.26</version>
    <scope>runtime</scope>
  </dependency>

  <!-- Optional dependency -->
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.32</version>
    <optional>true</optional>
  </dependency>

  <!-- Exclusion -->
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.0</version>
    <exclusions>
      <exclusion>
        <groupId>javax.persistence</groupId>
        <artifactId>javax.persistence-api</artifactId>
      </exclusion>
    </exclusions>
  </dependency>

</dependencies>
```

**Dependency Management:**
```xml
<!-- Parent POM: Centralize versions -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-framework-bom</artifactId>
      <version>5.3.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<!-- Child POM: No version needed -->
<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
  </dependency>
</dependencies>
```

## 🔄 Build Lifecycle Phases

| # | Phase | Description |
|---|---|---|
| 1 | `validate` | Validate project is correct |
| 2 | `compile` | Compile source code |
| 3 | `test-compile` | Compile test sources |
| 4 | `test` | Run tests |
| 5 | `package` | Create JAR/WAR |
| 6 | `verify` | Verify artifact quality |
| 7 | `install` | Install to local repo |
| 8 | `deploy` | Deploy to remote repo |

## 🔌 Plugin Configuration

**Compiler Plugin:**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.8.1</version>
  <configuration>
    <source>11</source>
    <target>11</target>
    <encoding>UTF-8</encoding>
  </configuration>
</plugin>
```

**Surefire Plugin (Testing):**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.22.2</version>
  <configuration>
    <includes>
      <include>**/*Test.java</include>
    </includes>
  </configuration>
</plugin>
```

**JAR Plugin:**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.2.0</version>
  <configuration>
    <archive>
      <manifest>
        <mainClass>com.example.Main</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
```

**Shade Plugin (Fat JAR):**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>3.2.4</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

## 🔧 Maven Properties

**Common Properties:**
```xml
<properties>
  <!-- Compiler properties -->
  <maven.compiler.source>11</maven.compiler.source>
  <maven.compiler.target>11</maven.compiler.target>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

  <!-- Dependency versions -->
  <spring.version>5.3.0</spring.version>
  <junit.version>4.13.2</junit.version>
  <lombok.version>1.18.20</lombok.version>
</properties>
```

**Using Properties:**
```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>${spring.version}</version>
</dependency>

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>${junit.version}</version>
</dependency>
```

## 🌐 Maven Profiles

**Profile Configuration:**
```xml
<profiles>
  <profile>
    <id>dev</id>
    <properties>
      <env>development</env>
    </properties>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
  </profile>
  <profile>
    <id>prod</id>
    <properties>
      <env>production</env>
    </properties>
  </profile>
</profiles>
```

**Activate Profiles:**
```bash
# Activate by ID
mvn clean install -Pdev

# Activate multiple
mvn clean install -Pdev,prod

# Check active profiles
mvn help:active-profiles
```

## 📚 Repository Configuration

**Repositories in POM:**
```xml
<repositories>
  <repository>
    <id>central</id>
    <url>https://repo.maven.apache.org/maven2</url>
  </repository>
  <repository>
    <id>spring-milestone</id>
    <url>https://repo.spring.io/milestone</url>
  </repository>
</repositories>
```

**settings.xml Configuration:**
```xml
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>password</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus.example.com:8081/nexus/content/groups/public</url>
    </mirror>
  </mirrors>

  <proxies>
    <proxy>
      <id>corporate-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.example.com</host>
      <port>8080</port>
    </proxy>
  </proxies>
</settings>
```

## 🧩 Multi-Module Projects

**Multi-Module Structure:**
```
my-app/
├── pom.xml (parent)
├── module-api/
│   └── pom.xml
├── module-core/
│   └── pom.xml
└── module-web/
    └── pom.xml
```

**Parent POM for Multi-Module:**
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app-parent</artifactId>
  <version>1.0.0</version>
  <packaging>pom</packaging>

  <modules>
    <module>module-api</module>
    <module>module-core</module>
    <module>module-web</module>
  </modules>
</project>
```

**Build Multi-Module:**
```bash
# Build all modules
mvn clean install

# Build specific module
mvn clean install -pl module-api

# Build module and dependencies
mvn clean install -pl module-api -am

# Build module and dependents
mvn clean install -pl module-api -amd
```

## ✅ Testing Configuration

**JUnit 4 Setup:**
```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13.2</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-core</artifactId>
  <version>3.12.4</version>
  <scope>test</scope>
</dependency>
```

**JUnit 5 Setup:**
```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-api</artifactId>
  <version>5.8.0</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-engine</artifactId>
  <version>5.8.0</version>
  <scope>test</scope>
</dependency>
```

## 🧰 Common Plugins Quick Reference

| Plugin | Purpose | Command |
|---|---|---|
| `maven-compiler-plugin` | Compile Java source code | `mvn compile` |
| `maven-surefire-plugin` | Run unit tests | `mvn test` / `mvn test -Dtest=ClassName` |
| `maven-jar-plugin` | Create JAR archive | `mvn package` |
| `maven-shade-plugin` | Create fat JAR with dependencies | `mvn clean shade:shade` |
| `maven-assembly-plugin` | Create distribution assemblies | `mvn assembly:single` |
| `maven-deploy-plugin` | Deploy artifact to repository | `mvn deploy` |
| `maven-release-plugin` | Release management | `mvn release:prepare` / `mvn release:perform` |
| `maven-javadoc-plugin` | Generate JavaDoc | `mvn javadoc:javadoc` |
| `maven-source-plugin` | Create source JAR | `mvn source:jar` |
| `jacoco-maven-plugin` | Code coverage analysis | `mvn jacoco:report` |
| `maven-checkstyle-plugin` | Code style verification | `mvn checkstyle:check` |
| `spotbugs-maven-plugin` | Find bugs in code | `mvn spotbugs:check` |

## 🌎 Environment Variables

| Variable | Example | Purpose |
|---|---|---|
| `MAVEN_HOME` | `/usr/local/maven` | Maven installation dir |
| `MAVEN_OPTS` | `-Xmx1024m -Xms512m` | JVM options for Maven |
| `JAVA_HOME` | `/usr/lib/jvm/java-11` | Java installation dir |

```bash
# Set Maven options for current session
export MAVEN_OPTS="-Xmx1024m -Xms512m"

# Set JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-11
```

## 🩺 Quick Troubleshooting Guide

| Issue | Solution |
|---|---|
| Dependency not found | `mvn dependency:tree -Dverbose` |
| Build slow | `mvn clean install -T 1C` |
| Tests fail randomly | Run again or isolate test |
| Plugin not found | Check plugin groupId/artifactId |
| Memory issues | `export MAVEN_OPTS="-Xmx2048m"` |
| Cannot find repo | Check `settings.xml`, proxy |
| Version conflict | Use `dependencyManagement` |
| Compilation fails | `mvn clean compile -X` |

## ✅ Best Practices Cheatsheet

**✔️ DO These:**
- Use DRY principle in POMs
- Centralize version management
- Lock versions in production
- Use parent POMs effectively
- Declare all dependencies explicitly
- Use profiles for environments
- Document complex configurations
- Test before deploying
- Keep POMs clean and organized
- Use meaningful artifact names

**⚠️ AVOID These:**
- Hardcoding versions
- Using version ranges
- Circular dependencies
- Uncommitted POMs
- Ignoring test failures
- Complex POM inheritance
- Repository config in POM
- Unversioned artifacts
- Ignoring deprecation warnings
- Manual dependency management

## 🏷️ Artifact Coordinates

Format: `groupId:artifactId:version:packaging:classifier`

Example:
```
org.springframework:spring-core:5.3.0:jar
```

Components:
- **groupId**: Organization identifier (`com.example`)
- **artifactId**: Project name (`my-app`)
- **version**: Release version (`1.0.0`)
- **packaging**: Type of file (`jar`, `war`, `pom`)
- **classifier**: Variant (`sources`, `javadoc`, `windows-x64`)

## 🔤 Maven Naming Conventions

| Element | Convention | Example |
|---|---|---|
| `groupId` | Reverse domain | `com.example.team` |
| `artifactId` | Lowercase with hyphens | `my-core-library` |
| `version` | Major.Minor.Patch | `1.2.3` / `1.0.0-SNAPSHOT` |
| Package | Lowercase, dots | `com.example.core` |
| Class | CamelCase, starts uppercase | `UserService` |
| Method | camelCase, starts lowercase | `getUserById()` |

## 🔢 Version Numbering

**Semantic Versioning:**

Format: `MAJOR.MINOR.PATCH`

Examples:
- `1.0.0` = Initial release
- `1.1.0` = Minor features added
- `1.1.1` = Bug fixes
- `2.0.0` = Major breaking changes

Qualifiers:
- `1.0.0-SNAPSHOT` = Development
- `1.0.0-alpha` = Early preview
- `1.0.0-beta` = Later preview
- `1.0.0-rc1` = Release candidate
- `1.0.0-final` = Final release

## 🎓 Interview Quick Guide

**Key Concepts to Know:**
- Maven Lifecycle: Clean, Build, Package, Deploy
- Dependency Scopes: compile, test, runtime, provided, optional
- POM Structure: modelVersion, groupId, artifactId, version
- Parent POM: Inheritance and aggregation
- DependencyManagement: Centralize versions
- Properties: `${variable}` syntax for reuse
- Plugins: Customize build behavior
- Profiles: Environment-specific configs
- Multi-module: Multiple projects in one
- Repositories: Central, remote, local `~/.m2`

## ⚡ Quick Command Reference

```bash
# Create new project
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app

# Build phases
mvn compile     # Compile
mvn test        # Run tests
mvn package     # Create JAR
mvn install     # Install locally
mvn deploy      # Deploy to repo

# Useful commands
mvn clean install               # Full clean build
mvn clean install -DskipTests   # Skip tests
mvn -U clean install            # Force update
mvn -X clean install            # Debug mode
mvn -o clean install            # Offline mode

# Dependency tools
mvn dependency:tree      # Show dependency tree
mvn dependency:analyze   # Analyze dependencies
mvn dependency:resolve   # Download all deps

# Information
mvn help:active-profiles           # Show active profiles
mvn help:effective-pom             # Show effective POM
mvn help:describe -Dplugin=compiler  # Plugin info
```

## 📋 Quick Reference Table

| Task | Command |
|---|---|
| Full build | `mvn clean install` |
| Skip tests | `mvn clean install -DskipTests` |
| Run one test | `mvn test -Dtest=ClassName` |
| Compile only | `mvn compile` |
| Package only | `mvn package` |
| Show dependencies | `mvn dependency:tree` |
| Parallel build | `mvn clean install -T 1C` |
| Offline build | `mvn clean install -o` |
| Check plugins | `mvn help:active-profiles` |
| Deploy artifact | `mvn deploy` |

## 🏗️ Advanced Configuration Patterns

**Build Extensions:**
```xml
<extensions>
  <extension>
    <groupId>org.apache.maven.wagon</groupId>
    <artifactId>wagon-ssh</artifactId>
    <version>3.4.2</version>
  </extension>
  <extension>
    <groupId>org.apache.maven.wagon</groupId>
    <artifactId>wagon-webdav-jackrabbit</artifactId>
    <version>3.4.2</version>
  </extension>
</extensions>
```

**Build Helpers:**
```xml
<!-- Add source directories -->
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>build-helper-maven-plugin</artifactId>
  <version>3.2.0</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>add-source</goal>
      </goals>
      <configuration>
        <sources>
          <source>${basedir}/src/main/groovy</source>
        </sources>
      </configuration>
    </execution>
  </executions>
</plugin>
```

**Skip Integration Tests:**
```xml
<!-- Skip IT tests -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>2.22.2</version>
  <configuration>
    <skipITs>false</skipITs>
  </configuration>
</plugin>
```

## 🐳 Docker Integration

**Dockerfile Location:**
```
project-root/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
└── src/
```

**Dockerfile Example:**
```dockerfile
FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Docker Maven Plugin:**
```xml
<plugin>
  <groupId>io.fabric8</groupId>
  <artifactId>docker-maven-plugin</artifactId>
  <version>0.37.0</version>
  <configuration>
    <images>
      <image>
        <name>my-company/my-app:${project.version}</name>
        <build>
          <dockerFile>${project.basedir}/Dockerfile</dockerFile>
        </build>
      </image>
    </images>
  </configuration>
</plugin>
```

## 🔗 Git Integration Commands

```bash
# Initialize Git
git init

# Clone Maven project
git clone https://github.com/user/project.git

# Maven release with Git
mvn release:prepare
mvn release:perform

# Rollback failed release
mvn release:rollback
```

## 🚦 Performance Tuning Tips

| Technique | Command | Benefit |
|---|---|---|
| Parallel builds | `mvn -T 1C` | 30-50% faster |
| Skip tests | `mvn -DskipTests` | 60-80% faster |
| Offline mode | `mvn -o` | Instant on cached artifacts |
| Incremental compile | Plugin config | Much faster |
| Heap increase | `export MAVEN_OPTS="-Xmx2048m"` | Prevents OOM |
| Local repository caching | Reuse artifacts | Faster resolution |

## 🔁 Continuous Integration Setup

**Jenkins Pipeline Example:**
```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/user/repo.git'
      }
    }
    stage('Build') {
      steps {
        sh 'mvn clean install'
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Deploy') {
      steps {
        sh 'mvn deploy'
      }
    }
  }
}
```

**GitLab CI Example:**
```yaml
image: maven:3.8.1-openjdk-11

stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - mvn compile

test:
  stage: test
  script:
    - mvn test

deploy:
  stage: deploy
  script:
    - mvn deploy
  only:
    - master
```

## 🌍 Environment Specific Builds

**Three Environment Example:**
```xml
<profiles>
  <profile>
    <id>development</id>
    <activation><activeByDefault>true</activeByDefault></activation>
    <properties>
      <env>dev</env>
      <db.host>localhost</db.host>
      <db.port>5432</db.port>
    </properties>
  </profile>
  <profile>
    <id>staging</id>
    <properties>
      <env>staging</env>
      <db.host>staging-db.internal</db.host>
      <db.port>5432</db.port>
    </properties>
  </profile>
  <profile>
    <id>production</id>
    <properties>
      <env>prod</env>
      <db.host>prod-db.internal</db.host>
      <db.port>5432</db.port>
    </properties>
  </profile>
</profiles>
```

**Build commands:**
```bash
mvn clean install               # Uses dev profile
mvn clean install -Pstaging     # Uses staging
mvn clean install -Pproduction  # Uses production
```

## 🔐 Security and Credentials

**Encrypt Password in Settings:**
```bash
# Create master password
mvn --encrypt-master-password "my-password"

# Create server password
mvn --encrypt-password "server-password"

# Copy encrypted values to settings.xml
```

**Secure Settings Example:**
```xml
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>{xxx...encrypted...xxx}</password>
    </server>
  </servers>
</settings>
```

## 📖 Effective POM Command

```bash
# Show final POM with all defaults
mvn help:effective-pom

# Save to file
mvn help:effective-pom > effective-pom.xml

# Show for specific module
mvn help:effective-pom -pl :module-name
```

## ⚔️ Dependency Conflict Resolution

**Nearest Wins Strategy:**

If a dependency appears at multiple depths, Maven uses the one closest to the project.

Example:
```
Project -> A (1.0) -> B -> C (1.0)
Project -> D -> C (2.0)

Result: C 1.0 is used (closer to project)
```

**Exclusion Example:**
```xml
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-core</artifactId>
  <version>5.6.0</version>
  <exclusions>
    <exclusion>
      <groupId>javax.persistence</groupId>
      <artifactId>javax.persistence-api</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

## 📸 Snapshot vs Release

| Aspect | Release (`1.0.0`) | Snapshot (`1.0.0-SNAPSHOT`) |
|---|---|---|
| Version format | Fixed | Mutable |
| Re-download | Once (cached) | Every build with `-U` |
| Use in prod | Yes | No |
| Repository | Releases | Snapshots |
| Stability | Stable | Unstable |

## 🧭 Maven Coordinates Details

**Full Coordinate Example:**
```
org.springframework.boot:spring-boot-starter-web:2.5.0
```

Breaking down:
- `org.springframework.boot` = groupId (organization)
- `spring-boot-starter-web` = artifactId (project name)
- `2.5.0` = version (release version)

Additional optional parts:
- `jar` = packaging (implicit if not specified)
- `sources` = classifier (optional, for variants)

## 🗄️ Repository Types

| Type | Location | Purpose |
|---|---|---|
| Local | `~/.m2/repository` | Personal cache |
| Remote | Team/Company Nexus | Shared team artifacts |
| Central | `repo.maven.apache.org` | Public worldwide artifacts |
| Mirror | Any URL | Speed up downloads |

## 🎯 Plugin Goals vs Phases

**Understanding Plugin Execution:**
- **PHASES**: Abstract steps (compile, test, package)
- **GOALS**: Plugin-specific tasks

Binding goal to phase:
```
mvn compile  ->  runs compile phase  ->  Surefire compile:compile goal executes
```

Multiple goals per phase possible (package phase example):
- `jar:jar` (create JAR)
- `source:jar-no-fork` (create source JAR)
- `javadoc:jar` (create Javadoc JAR)

## 🏛️ Maven Archetypes

```bash
# List available archetypes
mvn archetype:list

# Create project from archetype
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart

# Popular archetypes:
maven-archetype-quickstart    # Simple JAR
maven-archetype-webapp        # Web application
maven-archetype-j2ee-simple   # J2EE application
maven-archetype-site          # Documentation
```

## 📁 File Structure Overview

**Standard Maven structure:**
```
src/
  main/
    java/          # Java source code
    resources/     # Properties, XML, etc
    webapp/        # Web content (for WAR)
  test/
    java/          # Test code
    resources/     # Test resources
target/             # Build output
pom.xml             # Project descriptor
```

## 🚀 Deploy Local and Remote

```bash
# Install to local repository (~/.m2)
mvn install

# Deploy to remote repository (requires <distributionManagement> in POM)
mvn deploy

# Deploy specific artifact
mvn deploy:deploy-file -DgroupId=com.example -DartifactId=my-artifact -Dversion=1.0.0 -Dfile=my-artifact-1.0.0.jar -Durl=http://nexus.example.com/repository/releases -DrepositoryId=nexus
```

## ⚙️ Generated Sources

**Handling Generated Code:**
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>build-helper-maven-plugin</artifactId>
      <version>3.2.0</version>
      <executions>
        <execution>
          <phase>generate-sources</phase>
          <goals>
            <goal>add-source</goal>
          </goals>
          <configuration>
            <sources>
              <source>${project.build.directory}/generated-sources/jaxb</source>
            </sources>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

## 🧹 Code Quality Tools Integration

**SonarQube Integration:**
```bash
# Run analysis with SonarQube
mvn clean install -Dsonar.projectKey=my-project -Dsonar.sources=src -Dsonar.host.url=http://sonarqube.example.com -Dsonar.login=token
```

**Checkstyle Plugin:**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.1.2</version>
  <configuration>
    <configLocation>google_checks.xml</configLocation>
  </configuration>
</plugin>
```

## 💡 Final Cheatsheet Tips

**Remember These Critical Commands:**
- `mvn clean install` — Most used command
- `mvn dependency:tree` — Debug dependencies
- `mvn -X` — Debug mode for troubleshooting
- `mvn -DskipTests` — Fast builds without tests
- `mvn -T 1C` — Parallel builds for speed
- `mvn help:effective-pom` — See final configuration
- `mvn -pl :module` — Build specific module
- `mvn -U` — Force dependency update

---
*Source: adapted from the Apache Maven cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
