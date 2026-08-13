# Docker Cheatsheet

Complete reference guide for Docker containerization, from basics to advanced production patterns.

## 1. Installation & Setup

### Installation on Ubuntu/Debian

```bash
# Update package manager
sudo apt-get update && sudo apt-get upgrade -y

# Install dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify installation
docker --version
docker run hello-world

# Add user to docker group
sudo usermod -aG docker $USER
```

### Installation on macOS & Windows

macOS - Using Homebrew:

```bash
brew install --cask docker
```

macOS - Using DMG:

Download from: https://www.docker.com/products/docker-desktop

Windows - Using Chocolatey:

```powershell
choco install docker-desktop
```

Windows - Using Installer:

Download from: https://www.docker.com/products/docker-desktop

### Initial Configuration

```bash
# Check Docker daemon status
sudo systemctl status docker

# Start Docker daemon
sudo systemctl start docker

# Enable Docker on boot
sudo systemctl enable docker

# Configure Docker daemon
sudo nano /etc/docker/daemon.json

# Verify configuration
docker info

# Test Docker installation
docker run hello-world
```

> **Setup Complete!** Your Docker installation is ready when `docker run hello-world` displays a success message.

## 2. Core Docker Commands

### Image Management Commands

| Command | Description | Example |
|---|---|---|
| `docker images` | List all local images | `docker images` |
| `docker pull` | Pull image from registry | `docker pull ubuntu:20.04` |
| `docker build` | Build image from Dockerfile | `docker build -t myapp:1.0 .` |
| `docker tag` | Tag image | `docker tag myapp:1.0 myrepo/myapp:1.0` |
| `docker push` | Push image to registry | `docker push myrepo/myapp:1.0` |
| `docker rmi` | Remove image | `docker rmi ubuntu:20.04` |
| `docker search` | Search Docker Hub | `docker search nginx` |
| `docker inspect` | Get image details | `docker inspect ubuntu:20.04` |

### Container Lifecycle Commands

| Command | Description | Example |
|---|---|---|
| `docker run` | Create and run container | `docker run -d --name app ubuntu:20.04` |
| `docker ps` | List running containers | `docker ps -a` |
| `docker start` | Start stopped container | `docker start container_id` |
| `docker stop` | Stop running container | `docker stop container_id` |
| `docker restart` | Restart container | `docker restart container_id` |
| `docker kill` | Force kill container | `docker kill container_id` |
| `docker rm` | Remove container | `docker rm container_id` |
| `docker pause` | Pause container | `docker pause container_id` |

## 3. Dockerfile Essentials

### Complete Dockerfile Example

```dockerfile
# Multi-stage production Dockerfile
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci --only=production

# Runtime stage
FROM alpine:3.18
RUN apk add --no-cache nodejs
RUN addgroup -g 1000 appuser && adduser -D -u 1000 -G appuser appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appuser /build/node_modules ./node_modules
COPY --chown=appuser:appuser . .
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"
CMD ["node", "app.js"]
```

### Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|---|---|---|
| `FROM` | Set base image | `FROM ubuntu:20.04` |
| `RUN` | Execute command | `RUN apt-get install -y curl` |
| `COPY` / `ADD` | Copy files | `COPY app.js /app/` |
| `WORKDIR` | Set working dir | `WORKDIR /app` |
| `ENV` | Set environment | `ENV NODE_ENV=production` |
| `EXPOSE` | Document port | `EXPOSE 3000` |
| `CMD` | Default command | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Override command | `ENTRYPOINT ["npm", "start"]` |
| `USER` | Set user | `USER appuser` |
| `VOLUME` | Create mount | `VOLUME ["/data"]` |
| `LABEL` | Add metadata | `LABEL version="1.0"` |
| `HEALTHCHECK` | Health status | `HEALTHCHECK CMD curl localhost` |

## 4. Container Running & Operations

### Run Container Variations

Basic run operations:

```bash
docker run -d --name myapp image:tag
docker run -it image:tag /bin/bash
docker run --rm image:tag command
```

Port mapping:

```bash
docker run -p 8080:3000 image:tag
docker run -p 127.0.0.1:8080:3000 image:tag
```

Environment variables:

```bash
docker run -e NODE_ENV=production image:tag
docker run --env-file .env image:tag
```

Volume mounting:

```bash
docker run -v /host:/container image:tag
docker run -v myvolume:/data image:tag
```

Resource limits:

```bash
docker run --memory=512m --cpus=1.0 image:tag
docker run --memory-swap=1g image:tag
```

Network & hostname:

```bash
docker run --network mynet image:tag
docker run --hostname myhost image:tag
```

Restart policies:

```bash
docker run --restart=always image:tag
docker run --restart=on-failure:5 image:tag
```

### Container Interaction Commands

```bash
# View logs
docker logs container_id
docker logs -f container_id          # Follow logs
docker logs --tail 50 container_id   # Last 50 lines

# Execute commands
docker exec -it container_id /bin/bash
docker exec container_id whoami
docker exec container_id ps aux

# Copy files
docker cp container_id:/path/file ./local
docker cp ./local/file container_id:/path/

# View container info
docker inspect container_id
docker top container_id
docker stats container_id
```

## 5. Docker Networking

### Network Management Commands

| Command | Purpose | Example |
|---|---|---|
| `docker network create` | Create network | `docker network create mynet` |
| `docker network ls` | List networks | `docker network ls` |
| `docker network inspect` | Get network details | `docker network inspect mynet` |
| `docker network connect` | Connect container to network | `docker network connect mynet container` |
| `docker network disconnect` | Disconnect container | `docker network disconnect mynet container` |
| `docker network rm` | Remove network | `docker network rm mynet` |
| `docker network prune` | Remove unused networks | `docker network prune` |

### Port Mapping & DNS

Port mapping syntax: `docker run -p [HOST_IP]:[HOST_PORT]:[CONTAINER_PORT]/[PROTOCOL]`

```bash
# Map port 8080 to container port 3000
docker run -p 8080:3000 image:tag

# Map to specific host interface
docker run -p 127.0.0.1:8080:3000 image:tag

# Map UDP port
docker run -p 8080:3000/udp image:tag

# Container name DNS resolution (on custom networks)
docker network create mynet
docker run -d --network mynet --name db postgres:14
docker run -d --network mynet --name app myapp:1.0
# In app: connect to postgresql://db:5432
```

## 6. Volumes & Data Persistence

### Volume Commands

| Command | Description | Example |
|---|---|---|
| `docker volume create` | Create named volume | `docker volume create mydata` |
| `docker volume ls` | List volumes | `docker volume ls` |
| `docker volume inspect` | Get volume details | `docker volume inspect mydata` |
| `docker volume rm` | Remove volume | `docker volume rm mydata` |
| `docker volume prune` | Remove unused volumes | `docker volume prune` |

### Volume Usage Examples

```bash
# Named volume
docker run -d -v mydata:/data image:tag

# Bind mount
docker run -d -v /host/path:/container/path image:tag

# Read-only volume
docker run -d -v mydata:/data:ro image:tag

# Multiple volumes
docker run -d \
  -v mydata:/data \
  -v logs:/var/log \
  -v /host/config:/etc/config:ro \
  image:tag

# Volume for database
docker volume create postgres_data
docker run -d \
  -v postgres_data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:14
```

## 7. Docker Compose Mastery

### Complete docker-compose.yml Example

```yaml
version: '3.8'
services:
  web:
    build: .
    image: myapp:1.0
    container_name: myapp_web
    ports:
      - "8080:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://user:pass@db:5432/mydb
      REDIS_URL: redis://cache:6379
    volumes:
      - ./src:/app/src
      - app-logs:/app/logs
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
  db:
    image: postgres:14-alpine
    container_name: myapp_db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
  cache:
    image: redis:7-alpine
    container_name: myapp_cache
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
  app-logs:
networks:
  backend:
    driver: bridge
```

### Docker Compose Commands

| Command | Description | Example |
|---|---|---|
| `docker-compose up` | Start services | `docker-compose up -d` |
| `docker-compose down` | Stop services | `docker-compose down` |
| `docker-compose build` | Build images | `docker-compose build` |
| `docker-compose logs` | View logs | `docker-compose logs -f web` |
| `docker-compose ps` | List services | `docker-compose ps` |
| `docker-compose exec` | Execute command | `docker-compose exec web bash` |
| `docker-compose restart` | Restart services | `docker-compose restart` |
| `docker-compose scale` (via `up --scale`) | Scale service | `docker-compose up -d --scale web=3` |

## 8. Image Building & Optimization

### Build Commands & Options

```bash
# Basic build
docker build -t myapp:1.0 .

# Build with arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build without cache
docker build --no-cache -t myapp:1.0 .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build multi-stage
docker build --target production -t myapp:1.0 .

# Build and tag multiple
docker build -t myapp:1.0 -t myapp:latest .

# View build history
docker history myapp:1.0

# Save image to file
docker save myapp:1.0 > myapp.tar

# Load image from file
docker load < myapp.tar
```

### Image Optimization Best Practices

`.dockerignore` example:

```
.git
.gitignore
node_modules
npm-debug.log
.env
.DS_Store
.vscode
.idea
dist
build
coverage
*.md
LICENSE
```

Checklist:

- Use minimal base images (alpine, slim)
- Combine `RUN` commands to reduce layers
- Use multi-stage builds for smaller final images
- Remove unnecessary files and dependencies
- Place frequently changing layers near the end
- Use `.dockerignore` to exclude files
- Don't use the `latest` tag in production
- Use specific base image versions
- Run as a non-root user
- Remove package manager caches

## 9. Registry Operations

### Docker Hub Operations

```bash
# Login to Docker Hub
docker login
docker login -u username -p password
docker login -u username --password-stdin < password.txt

# Search images
docker search ubuntu
docker search nginx --limit 25

# Pull image
docker pull ubuntu:20.04
docker pull myusername/myapp:1.0

# Tag image for push
docker tag myapp:1.0 myusername/myapp:1.0
docker tag myapp:1.0 myusername/myapp:latest

# Push to Docker Hub
docker push myusername/myapp:1.0
docker push myusername/myapp:latest

# Logout
docker logout
```

### Private Registry Setup

```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for private registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Push to private registry
docker push localhost:5000/myapp:1.0

# Pull from private registry
docker pull localhost:5000/myapp:1.0

# Registry with persistent storage
docker run -d \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  --name registry \
  registry:2
```

## 10. Debugging & Monitoring

### Logging Commands

| Command | Purpose | Example |
|---|---|---|
| `docker logs` | View container logs | `docker logs container_id` |
| `docker logs -f` | Follow logs | `docker logs -f container_id` |
| `--tail` | Show last N lines | `docker logs --tail 100 container_id` |
| `--since` | Show since timestamp | `docker logs --since 10m container_id` |
| `--timestamps` | Add timestamps | `docker logs --timestamps container_id` |

### Monitoring & Inspection Commands

```bash
# View running processes
docker top container_id

# View container statistics
docker stats
docker stats container_id
docker stats --no-stream

# Inspect container
docker inspect container_id
docker inspect --format='{{.State.Status}}' container_id

# View filesystem changes
docker diff container_id

# List events
docker events
docker events --filter type=container
docker events --filter event=die

# View image history
docker history myapp:1.0
```

### Debugging Techniques

Debugging tips:

- Check logs first: `docker logs container_id`
- Inspect container: `docker inspect container_id`
- Execute interactive shell: `docker exec -it container_id /bin/bash`
- Check processes: `docker top container_id`
- View resource usage: `docker stats container_id`
- Commit for inspection: `docker commit container_id debug:1.0`

## 11. Security Best Practices

### Security Do's

- Use specific base image versions
- Run containers as a non-root user
- Use read-only filesystems where possible
- Implement health checks
- Use resource limits
- Scan images for vulnerabilities
- Keep the Docker daemon updated
- Use environment variables for secrets
- Enable user namespaces
- Implement network policies

### Security Don'ts

- Don't use the `latest` tag in production
- Don't run containers as root
- Don't store secrets in the Dockerfile
- Don't mount the host filesystem unnecessarily
- Don't run privileged containers
- Don't disable security features
- Don't use default credentials
- Don't skip image scanning
- Don't hardcode sensitive data
- Don't expose unnecessary ports

### Security Commands

```bash
# Scan image for vulnerabilities
docker scan myapp:1.0

# Run with limited capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE image:tag

# Run with read-only filesystem
docker run --read-only --tmpfs /tmp image:tag

# Run with resource limits
docker run --memory=512m --cpus=1.0 image:tag

# Run as specific user
docker run -u 1000:1000 image:tag

# Enable user namespace mapping
docker daemon --userns-remap=default
```

## 12. Deployment Architecture Patterns

### Pattern 1: Multi-Container Application

Architecture overview:

```
Frontend → API Gateway → Microservices → Databases
```

```bash
# Components:
# - Web frontend (port 3000)
# - API gateway (port 5000)
# - User service (internal)
# - Product service (internal)
# - PostgreSQL database
# - Redis cache
```

### Pattern 2: Microservices with Service Mesh

Services communicate through a service mesh for:

- Service discovery
- Load balancing
- Circuit breaking
- Observability

### Pattern 3: Event-Driven Architecture

Services communicate through message queues:

```bash
# Components:
# - Event producers
# - Message broker (RabbitMQ/Kafka)
# - Event consumers
# - Event persistence
```

## 13. Health Checks & Self-Healing

### Health Check Configuration

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
```

```yaml
# In docker-compose.yml
services:
  app:
    image: myapp:1.0
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      start_period: 5s
      retries: 3
```

```bash
# Check health status
docker inspect --format='{{.State.Health.Status}}' container_id
docker inspect --format='{{json .State.Health}}' container_id | jq
```

### Health Check Endpoints

| Endpoint | Purpose | Returns |
|---|---|---|
| `/health` | Basic health check | `200 OK` |
| `/health/live` | Liveness probe | `200` if alive |
| `/health/ready` | Readiness probe | `200` if ready |
| `/health/startup` | Startup probe | `200` if started |

## 14. Troubleshooting & Common Issues

### Container exits immediately

```bash
# Check logs
docker logs container_id

# Run interactively
docker run -it image:tag /bin/bash

# Inspect exit code
docker inspect container_id | grep ExitCode
```

### Port already in use

```bash
# Find container using port
docker ps | grep :8080
lsof -i :8080

# Use different port
docker run -p 8081:3000 image:tag
```

### Out of disk space

```bash
# Check disk usage
docker system df

# Clean up unused resources
docker system prune -a --volumes
docker image prune -a
docker volume prune
```

### Memory issues

```bash
# Monitor memory
docker stats container_id

# Set memory limits
docker run --memory=512m --memory-swap=1g image:tag

# Check memory usage
free -h
docker top container_id
```

### Network connectivity

```bash
# Test DNS resolution
docker exec container_id nslookup service_name

# Test connectivity
docker exec container_id ping service_name
docker exec container_id curl http://service:port

# Check network configuration
docker network inspect mynetwork
```

### Permission denied errors

```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Run with specific user
docker run -u 1000:1000 image:tag

# Check file permissions
docker exec container_id ls -la /app
```

## 15. Quick Reference Cards

### Essential Docker Commands Summary

| Category | Command | Use Case |
|---|---|---|
| Images | `docker images` | List all images |
| Images | `docker pull image:tag` | Download image |
| Containers | `docker run -d image:tag` | Run container |
| Containers | `docker ps -a` | List containers |
| Logs | `docker logs -f container` | Follow logs |
| Build | `docker build -t tag .` | Build image |
| Network | `docker network create name` | Create network |
| Volume | `docker volume create name` | Create volume |
| Compose | `docker-compose up -d` | Start services |
| Cleanup | `docker system prune -a` | Remove unused |
| Exec | `docker exec -it container bash` | Interactive shell |
| Stats | `docker stats` | Monitor resources |

## 16. Performance Optimization & Production Tips

### Resource Management

```bash
# Set memory limits
docker run --memory=512m --memory-swap=1g image:tag

# Set CPU limits
docker run --cpus=1.5 --cpu-shares=1024 image:tag

# Set block I/O limits
docker run --blkio-weight=300 image:tag
```

```yaml
# In docker-compose.yml
services:
  app:
    image: myapp:1.0
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### Dockerfile & Runtime Performance Tips

Production optimization checklist:

- Enable Docker daemon debug logging when needed
- Use tmpfs for temporary data
- Implement a proper logging strategy
- Monitor container metrics regularly
- Use health checks for automatic recovery
- Implement graceful shutdown
- Use connection pooling
- Implement caching strategies
- Load test before production
- Monitor resource utilization continuously
- Use `.dockerignore` to exclude unnecessary files
- Order Dockerfile instructions from least to most frequently changing
- Combine `RUN` commands to reduce layers
- Use multi-stage builds to reduce final image size
- Remove package manager caches
- Use minimal base images (alpine, slim)
- Avoid running unnecessary services
- Cache dependencies separately from code
- Use specific versions for base images
- Clean up temporary files immediately

---

*Source: adapted from the Docker cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
