# Podman Cheatsheet

Comprehensive quick reference for all Podman commands and operations, organized by category for easy navigation and lookup. Includes practical examples with real-world use cases, covers container/image/pod/network/volume management, security best practices, troubleshooting tips, Docker equivalent commands, and systemd integration / rootless operation specifics.

## ⚙️ Installation & Setup Commands

### Install Podman on Different Platforms

```bash
# Fedora/RHEL/CentOS
sudo dnf install -y podman

# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y podman

# macOS (using Homebrew)
brew install podman

# Windows (using WSL2)
wsl --install
sudo apt-get install -y podman
```

Verify installation:

```bash
podman --version
podman info
```

Check system compatibility:

```bash
podman system info
podman system connection list
```

### Initial Configuration

Set up rootless Podman for a regular user:

```bash
podman system migrate
```

Configure registries:

```bash
cat ~/.config/containers/registries.conf
```

Add a default registry to search (e.g. Docker Hub):

```bash
echo 'unqualified-search-registries = ["docker.io"]' >> \
  ~/.config/containers/registries.conf
```

Enable the Podman socket for Docker compatibility:

```bash
systemctl --user enable --now podman.socket
```

Set Docker environment variable for compatibility:

```bash
export DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock
```

Check the storage driver in use:

```bash
podman info --format '{{.Store.GraphDriverName}}'
```

## 📦 Container Lifecycle Management

### Running Containers

```bash
# Run container in foreground
podman run nginx:alpine

# Run container in detached mode (background)
podman run -d nginx:alpine

# Run with custom name
podman run -d --name my-nginx nginx:alpine

# Run with port mapping (host:container)
podman run -d -p 8080:80 --name web nginx:alpine

# Run with environment variables
podman run -d -e MYSQL_ROOT_PASSWORD=secret123 mysql:8

# Run with volume mount
podman run -d -v /host/path:/container/path:Z nginx:alpine
podman run -d -v my-volume:/data nginx:alpine

# Run with resource limits
podman run -d --memory=512m --cpus=1.5 nginx:alpine

# Run interactive container with TTY
podman run -it alpine sh

# Run container and remove after exit
podman run --rm alpine echo "Hello Podman"

# Run with custom network
podman run -d --network my-net nginx:alpine

# Run with multiple port mappings
podman run -d -p 8080:80 -p 8443:443 nginx:alpine

# Run with restart policy
podman run -d --restart=always nginx:alpine

# Run with health check
podman run -d \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  nginx:alpine

# Run with user specification
podman run -d --user 1000:1000 nginx:alpine

# Run with hostname
podman run -d --hostname my-container nginx:alpine

# Run with DNS settings
podman run -d --dns 8.8.8.8 --dns 8.8.4.4 nginx:alpine

# Run with working directory
podman run -d --workdir /app nginx:alpine

# Run with security options (SELinux)
podman run -d --security-opt label=level:s0:c100,c200 nginx:alpine

# Run privileged container (use with caution)
podman run -d --privileged nginx:alpine
```

### Container Status & Information

```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# List containers with custom format
podman ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# List only container IDs
podman ps -q

# Filter containers by status
podman ps --filter status=running
podman ps --filter status=exited

# Filter containers by name
podman ps --filter name=nginx

# Show last created container
podman ps -l

# Inspect container details
podman inspect my-nginx

# Get specific field from inspect
podman inspect --format '{{.State.Status}}' my-nginx
podman inspect --format '{{.NetworkSettings.IPAddress}}' my-nginx

# View container logs
podman logs my-nginx

# Follow logs in real-time
podman logs -f my-nginx

# Show last N lines of logs
podman logs --tail 50 my-nginx

# Show logs with timestamps
podman logs -t my-nginx

# Show container resource usage
podman stats my-nginx

# View stats for all containers (single snapshot)
podman stats --no-stream

# View container processes
podman top my-nginx

# View container port mappings
podman port my-nginx
```

### Container Control Operations

```bash
# Stop container gracefully
podman stop my-nginx

# Stop with custom timeout (default 10s)
podman stop -t 30 my-nginx

# Stop all running containers
podman stop $(podman ps -q)

# Kill container immediately (SIGKILL)
podman kill my-nginx

# Send custom signal to container
podman kill -s SIGHUP my-nginx

# Start stopped container
podman start my-nginx

# Restart container
podman restart my-nginx

# Pause container (freezes processes)
podman pause my-nginx

# Unpause container
podman unpause my-nginx

# Remove stopped container
podman rm my-nginx

# Force remove running container
podman rm -f my-nginx

# Remove all stopped containers
podman container prune

# Remove all containers (including running)
podman rm -f $(podman ps -aq)

# Rename container
podman rename old-name new-name

# Wait for container to stop
podman wait my-nginx

# Attach to running container
podman attach my-nginx

# Execute command in running container
podman exec my-nginx ls -la /etc

# Execute interactive shell in container
podman exec -it my-nginx /bin/bash

# Execute as specific user
podman exec -u root my-nginx whoami

# Copy files from container to host
podman cp my-nginx:/app/config.json ./config.json

# Copy files from host to container
podman cp ./local-file.txt my-nginx:/tmp/

# Export container filesystem to tar
podman export my-nginx > container-backup.tar

# Create new image from container changes
podman commit my-nginx custom-image:v1
```

## 🖼️ Image Management

### Image Operations

```bash
# Search for images
podman search nginx
podman search --filter is-official=true nginx
podman search --limit 10 nginx

# Pull image from registry
podman pull nginx:alpine

# Pull from specific registry
podman pull docker.io/library/nginx:latest
podman pull quay.io/podman/hello

# Pull all tags of an image
podman pull --all-tags nginx

# List local images
podman images

# List with detailed information
podman images --no-trunc

# List image IDs only
podman images -q

# Filter images
podman images --filter reference=nginx
podman images --filter dangling=true

# Show image history
podman history nginx:alpine

# Inspect image details
podman inspect nginx:alpine

# Get image size
podman inspect --format='{{.Size}}' nginx:alpine

# Tag image
podman tag nginx:alpine mynginx:v1

# Remove image
podman rmi nginx:alpine

# Force remove image
podman rmi -f nginx:alpine

# Remove unused images
podman image prune

# Remove all unused images (including tagged)
podman image prune -a

# Remove all images
podman rmi $(podman images -q)

# Save image to tar file
podman save -o nginx-backup.tar nginx:alpine

# Save multiple images
podman save -o multi-images.tar nginx:alpine redis:alpine

# Load image from tar file
podman load -i nginx-backup.tar

# Push image to registry
podman push mynginx:v1 docker.io/username/mynginx:v1

# Push to private registry
podman push --tls-verify=false myimage:v1 localhost:5000/myimage:v1
```

### Building Images

```bash
# Build image from Dockerfile (or Containerfile)
podman build -t myapp:v1 .

# Build with specific file
podman build -t myapp:v1 -f Dockerfile.prod .

# Build with build arguments
podman build --build-arg VERSION=1.2.3 -t myapp:v1 .

# Build without cache
podman build --no-cache -t myapp:v1 .

# Build a specific target (multi-stage)
podman build --target production -t myapp:v1 .

# Build with layer squashing
podman build --squash -t myapp:v1 .

# Build with different format
podman build --format docker -t myapp:v1 .

# Build with resource limits
podman build --memory 2g --cpus 2 -t myapp:v1 .

# Build and push in one command
podman build -t myapp:v1 . && podman push myapp:v1

# Show build progress
podman build --progress=plain -t myapp:v1 .
```

## 🎛️ Pod Management

### Pod Operations

```bash
# Create empty pod
podman pod create --name my-pod

# Create pod with port mapping
podman pod create --name web-pod -p 8080:80

# Create pod with custom network
podman pod create --name app-pod --network my-network

# Create pod with hostname
podman pod create --name db-pod --hostname database

# List all pods
podman pod ps
podman pod ps --ctr-names

# Inspect pod details
podman pod inspect my-pod

# Add container to existing pod
podman run -d --pod my-pod nginx:alpine
podman run -d --pod my-pod redis:alpine

# Start pod (starts all containers)
podman pod start my-pod

# Stop pod (stops all containers)
podman pod stop my-pod

# Restart pod
podman pod restart my-pod

# Pause pod (pauses all containers)
podman pod pause my-pod

# Unpause pod
podman pod unpause my-pod

# Remove pod (must stop first or use -f)
podman pod rm my-pod

# Force remove pod
podman pod rm -f my-pod

# Remove all stopped pods
podman pod prune

# View pod logs
podman pod logs my-pod

# View stats for all containers in a pod
podman pod stats my-pod

# View processes in pod
podman pod top my-pod

# Kill pod
podman pod kill my-pod
```

### Kubernetes YAML Generation

```bash
# Generate Kubernetes YAML from pod
podman generate kube my-pod > my-pod.yaml

# Generate YAML including services
podman generate kube --service my-pod > my-pod-with-service.yaml
podman generate kube my-container > my-container.yaml

# Play (deploy) Kubernetes YAML
podman play kube my-pod.yaml

# Play with custom network
podman play kube --network my-net my-pod.yaml

# Remove resources from Kubernetes YAML
podman play kube --down my-pod.yaml
```

## 🌐 Network Management

### Network Operations

```bash
# List networks
podman network ls

# Create network
podman network create my-network

# Create network with subnet
podman network create --subnet 10.20.30.0/24 my-network

# Create network with subnet and gateway
podman network create --subnet 10.20.30.0/24 --gateway 10.20.30.1 my-network

# Create network with DNS
podman network create --dns 8.8.8.8 my-network

# Create network with driver
podman network create --driver bridge my-network

# Create internal network (no external access)
podman network create --internal my-internal-net

# Inspect network details
podman network inspect my-network

# Connect container to network
podman network connect my-network my-container

# Connect with custom IP
podman network connect --ip 10.20.30.50 my-network my-container

# Connect with alias
podman network connect --alias db my-network my-container

# Disconnect container from network
podman network disconnect my-network my-container

# Remove network
podman network rm my-network

# Remove unused networks
podman network prune

# Check if network exists
podman network exists my-network

# Reload network configuration
podman network reload my-container
```

## 💾 Volume Management

### Volume Operations

```bash
# List volumes
podman volume ls

# Create volume
podman volume create my-volume

# Create volume with driver options
podman volume create --opt type=tmpfs --opt device=tmpfs my-tmpfs-volume
podman volume create --label environment=production my-volume

# Inspect volume details
podman volume inspect my-volume

# Get volume mount point
podman volume inspect --format '{{.Mountpoint}}' my-volume

# Mount volume to container
podman run -d -v my-volume:/data nginx:alpine

# Mount with read-only
podman run -d -v my-volume:/data:ro nginx:alpine

# Mount with SELinux relabeling
podman run -d -v my-volume:/data:Z nginx:alpine

# Bind mount host directory
podman run -d -v /host/path:/container/path:Z nginx:alpine

# Remove volume
podman volume rm my-volume

# Force remove volume
podman volume rm -f my-volume

# Remove unused volumes
podman volume prune

# Export volume data
podman volume export my-volume --output volume-backup.tar

# Import volume data
podman volume import my-volume volume-backup.tar

# Check if volume exists
podman volume exists my-volume
```

## 🔒 Security & Authentication

### Registry Authentication

```bash
# Login to registry
podman login docker.io

# Login with credentials
podman login -u username -p password docker.io

# Login to private registry
podman login registry.example.com

# Login with password from file
cat password.txt | podman login -u username --password-stdin docker.io

# Logout from registry
podman logout docker.io

# Logout from all registries
podman logout --all

# View stored credentials
cat ~/.config/containers/auth.json
```

### Secrets Management

```bash
# Create secret from file
podman secret create my-secret ./secret-file.txt

# Create secret from a value
echo "my-secret-value" | podman secret create my-secret -

# List secrets
podman secret ls

# Inspect secret (metadata only)
podman secret inspect my-secret

# Use secret in container
podman run -d --secret my-secret nginx:alpine

# Use secret with custom target path
podman run -d --secret my-secret,target=/app/secret nginx:alpine

# Remove secret
podman secret rm my-secret
```

### Security Options

```bash
# Run with no new privileges
podman run -d --security-opt no-new-privileges nginx:alpine

# Run with custom SELinux label
podman run -d --security-opt label=level:s0:c100,c200 nginx:alpine

# Disable SELinux label
podman run -d --security-opt label=disable nginx:alpine

# Run with AppArmor profile
podman run -d --security-opt apparmor=docker-default nginx:alpine

# Run with seccomp profile
podman run -d --security-opt seccomp=./custom-seccomp.json nginx:alpine

# Run with read-only root filesystem
podman run -d --read-only nginx:alpine

# Add Linux capabilities
podman run -d --cap-add NET_ADMIN nginx:alpine
podman run -d --cap-drop ALL --cap-add NET_BIND_SERVICE nginx:alpine

# Set user namespace mapping
podman run -d --userns=keep-id nginx:alpine
```

## ⚙️ Systemd Integration

### Systemd Service Generation

```bash
# Generate systemd unit for container
podman generate systemd --new --files --name my-container

# Generate for pod
podman generate systemd --new --files --name my-pod

# Generate with custom restart policy
podman generate systemd --new --restart-policy=always --name my-container

# Generate with timeout
podman generate systemd --new --time 30 --name my-container

# Install systemd service
mkdir -p ~/.config/systemd/user/
mv container-my-container.service ~/.config/systemd/user/
systemctl --user daemon-reload

# Enable service
systemctl --user enable container-my-container.service

# Start service
systemctl --user start container-my-container.service

# Check service status
systemctl --user status container-my-container.service

# View service logs
journalctl --user -u container-my-container.service

# Enable lingering (keep services running without login)
loginctl enable-linger $USER

# Disable service
systemctl --user disable container-my-container.service

# Stop and remove service
systemctl --user stop container-my-container.service
systemctl --user disable container-my-container.service
rm ~/.config/systemd/user/container-my-container.service
systemctl --user daemon-reload
```

## 🔍 Troubleshooting & Debugging

### Health Checks & Diagnostics

```bash
# Check container health status
podman inspect --format='{{.State.Health.Status}}' my-container

# Run health check manually
podman healthcheck run my-container

# View system information
podman system info

# Check for system problems
podman system check

# View events in real-time
podman events

# Filter events by container
podman events --filter container=my-container

# View events in a specific time range
podman events --since 2024-01-01 --until 2024-01-31

# Show container diff (filesystem changes)
podman diff my-container

# View container resource constraints
podman inspect --format='{{.HostConfig.Memory}}' my-container

# Debug container startup issues
podman logs --tail 100 my-container
podman inspect my-container
podman events --filter container=my-container

# Test network connectivity
podman run --rm --network my-network alpine ping -c 3 another-container

# Verify DNS resolution
podman exec my-container nslookup google.com

# Check SELinux denials
sudo ausearch -m AVC -ts recent | grep podman
```

## 🧹 System Maintenance

### Cleanup & Optimization

```bash
# Remove stopped containers
podman container prune

# Remove unused images
podman image prune

# Remove unused volumes
podman volume prune

# Remove unused networks
podman network prune

# Remove all unused resources
podman system prune

# Also remove unused volumes
podman system prune --volumes

# Force removal without confirmation
podman system prune -f

# Remove everything (nuclear option)
podman system reset

# View disk usage
podman system df

# View detailed disk usage
podman system df -v

# Migrate Podman to new version
podman system migrate

# Renumber locks (fix lock issues)
podman system renumber
```

### Performance & Monitoring

```bash
# View container resource usage
podman stats

# View stats without streaming
podman stats --no-stream

# View stats for specific container
podman stats my-container

# Format stats output
podman stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Check container processes
podman top my-container

# Check with output columns
podman top my-container -o pid,user,comm

# Stream events continuously
podman events --stream

# Check for resource leaks
podman system df
podman ps -a --filter status=exited
```

## 🐳 Docker Compatibility

### Docker Alias & Socket

```bash
# Create Docker alias (add to ~/.bashrc or ~/.zshrc)
alias docker=podman

# Enable Podman socket for Docker CLI compatibility
systemctl --user enable --now podman.socket

# Set Docker environment variable
export DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock

# Verify Docker CLI works with Podman
docker ps
docker images

# Use docker-compose with Podman
pip install podman-compose
podman-compose up -d

# Emulate Docker socket
podman system service --time=0 unix:///tmp/podman.sock
```

## 🚀 Advanced Use Cases

### Multi-Container Applications

```bash
# Create pod for multi-tier app
podman pod create --name app-stack -p 8080:80

# Add database container
podman run -d --pod app-stack \
  --name db \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Add application container
podman run -d --pod app-stack \
  --name app \
  -e DATABASE_URL=postgresql://localhost/mydb \
  myapp:latest

# Add web server container
podman run -d --pod app-stack \
  --name web \
  nginx:alpine

# Generate Kubernetes manifest for entire stack
podman generate kube app-stack > app-stack.yaml

# Deploy to Kubernetes
kubectl apply -f app-stack.yaml
```

### CI/CD Integration

```bash
# Build image in CI pipeline
podman build --format docker -t myapp:${CI_COMMIT_SHA} .

# Tag for registry
podman tag myapp:${CI_COMMIT_SHA} registry.example.com/myapp:latest

# Push to registry
podman push registry.example.com/myapp:latest

# Run tests in container
podman run --rm myapp:${CI_COMMIT_SHA} npm test

# Security scan with Trivy
trivy image myapp:${CI_COMMIT_SHA}

# Deploy to production
podman pull registry.example.com/myapp:latest
podman stop prod-app || true
podman rm prod-app || true
podman run -d --name prod-app -p 80:8080 \
  registry.example.com/myapp:latest
```

## 📋 Quick Reference Tables

### Container States

| State | Description | Set by |
|---|---|---|
| `created` | Container created but not started | `podman create` |
| `running` | Container is currently executing | `podman start` |
| `paused` | Container processes are frozen | `podman pause` |
| `exited` | Container finished execution / process completed | `podman stop` |
| `dead` | Container failed to remove | Automatic on removal failure |

### Common Port Mappings

| Service | Port | Example |
|---|---|---|
| HTTP (Web Server) | 80 | `podman run -d -p 8080:80 nginx` |
| HTTPS | 443 | `podman run -d -p 8443:443 nginx` |
| PostgreSQL | 5432 | `podman run -d -p 5432:5432 postgres` |
| MySQL | 3306 | `podman run -d -p 3306:3306 mysql` |
| Redis | 6379 | `podman run -d -p 6379:6379 redis` |
| MongoDB | 27017 | `podman run -d -p 27017:27017 mongo` |
| RabbitMQ | 5672, 15672 | `podman run -d -p 5672:5672 -p 15672:15672 rabbitmq` |

### Volume Mount Options

| Option | Description | Example |
|---|---|---|
| `:z` (lowercase) | Shared SELinux label - multiple containers can access | `-v /path:/data:z` |
| `:Z` (uppercase) | Private SELinux label - only this container can access | `-v /path:/data:Z` |
| `:ro` | Read-only mount | `-v /path:/data:ro` |
| `:rw` | Read-write mount (default) | `-v /path:/data:rw` |
| `:U` | Chown to container user | `-v /path:/data:U` |

### Restart Policies

| Policy | Behavior | Use Case |
|---|---|---|
| `no` | Never restart (default) | One-time tasks, development |
| `on-failure` | Restart only on non-zero exit code | Tasks that may fail temporarily |
| `always` | Always restart regardless of exit code | Critical services, production apps |
| `unless-stopped` | Restart unless manually stopped | Long-running services |

## 💡 Best Practices & Tips

### ✅ Production Best Practices

- Always run containers in rootless mode for enhanced security
- Use specific image tags instead of `latest` for reproducibility
- Implement health checks for all production containers
- Set resource limits (CPU, memory) to prevent resource exhaustion
- Use named volumes instead of bind mounts for portability
- Enable automatic restart policies for critical services
- Implement proper logging and monitoring from day one
- Use multi-stage builds to minimize image size
- Scan images for vulnerabilities before deployment
- Keep containers stateless - store data in volumes or external services
- Use pods for tightly coupled containers that need to share a network namespace
- Implement proper secret management - never hardcode credentials
- Create separate networks for different application tiers
- Document all custom configurations and deployment procedures
- Test disaster recovery procedures regularly

### ⚡ Performance Tips

- Use `--pull=newer` to only pull images when updates are available
- Leverage build cache by ordering Dockerfile instructions from least to most frequently changed
- Use `.dockerignore` (or `.containerignore`) to exclude unnecessary files from the build context
- Clean up unused resources regularly with `podman system prune`
- Use Alpine-based images to reduce image size and attack surface
- Enable parallel pulls with `--max-parallel-downloads`
- Use local registries or caching proxies to speed up image pulls
- Mount volumes with appropriate SELinux options to avoid permission issues

### ⚠️ Common Pitfalls to Avoid

- Don't use `--privileged` unless absolutely necessary
- Avoid running containers as root when possible
- Don't store sensitive data in images - use secrets or environment variables
- Never expose unnecessary ports to the host
- Don't ignore SELinux denials - investigate and fix the root cause
- Avoid bind mounting entire host directories - be specific
- Don't mix rootful and rootless Podman contexts
- Never commit containers with sensitive data
- Don't forget to set health checks for long-running services
- Avoid using host network mode unless required

## ⚖️ Docker vs Podman Command Comparison

| Docker Command | Podman Equivalent | Notes |
|---|---|---|
| `docker run` | `podman run` | Identical syntax |
| `docker build` | `podman build` | Identical syntax |
| `docker ps` | `podman ps` | Identical syntax |
| `docker images` | `podman images` | Identical syntax |
| `docker-compose` | `podman-compose` | Requires separate installation |
| `docker stack` | `podman play kube` | Different approach - uses K8s YAML |
| Docker daemon | No daemon required | Podman is daemonless |
| `docker swarm` | Not supported | Use Kubernetes instead |

## ⚡ Quick Tips by Category

### 🏃 Speed Up Development

- Use `--rm` for throwaway containers
- Mount code with `-v $(pwd):/app:Z`
- Use `-it` for interactive debugging
- Create aliases for common commands

### 🔐 Enhance Security

- Always run rootless mode
- Use `--read-only` filesystem
- Drop unnecessary capabilities
- Scan images with Trivy or Clair

### 🐛 Debug Issues

- Check logs: `podman logs -f`
- Inspect: `podman inspect`
- Execute shell: `podman exec -it bash`
- Check events: `podman events`

### 🧽 Keep System Clean

- Weekly: `podman system prune`
- Check usage: `podman system df`
- Remove unused: `podman image prune -a`
- Nuclear option: `podman system reset`

## 🔑 Key Takeaways

- Podman provides a Docker-compatible CLI with enhanced security through rootless operation
- Most Docker commands work identically with Podman - just replace `docker` with `podman`
- Pods group containers with a shared network namespace, similar to Kubernetes pods
- `podman generate kube` enables seamless migration to Kubernetes environments
- Systemd integration provides production-grade container lifecycle management
- SELinux labeling (`z`, `Z`) is crucial for proper volume permissions on RHEL-based systems
- Health checks, resource limits, and restart policies are essential for production deployments
- Regular maintenance with prune commands keeps the system performant and clean
- Rootless mode eliminates the need for a Docker daemon and enhances security posture
- Network isolation and secrets management are critical for secure multi-tier applications

---

*Source: adapted from the Podman cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
