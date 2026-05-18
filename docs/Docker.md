# Comprehensive Docker Curriculum

Here’s a complete Docker curriculum from absolute beginner to expert, structured in progressive levels. It includes a structured curriculum, practical labs, commands reference, and a self-assessment checklist.

---

## 🟢 Level 1: Beginner (Foundations)

### Curriculum Topics

1. **What is Docker & Why Use It?**
   - Containers vs. Virtual Machines
   - Docker architecture (Client, Daemon, Registry)
   - Docker benefits (consistency, portability, efficiency)
2. **Installation & Setup**
   - Install Docker on Linux, Windows (WSL2), macOS
   - Docker Desktop vs. Docker Engine
   - Verify installation (`docker version`, `docker info`)
   - **The Docker Desktop Ecosystem**:
     - _Docker Dashboard GUI_: Managing containers, volumes, and images via the graphical interface.
     - _Docker Desktop Settings_: Resources (CPU/memory), Docker Engine JSON, Kubernetes toggle, WSL2 integration.
     - _Docker Scout Integration_: Reading vulnerability alerts, image recommendations, and quick remediation hints directly within Docker Desktop.
3. **Basic Docker Commands**
   - `docker pull`, `docker run`, `docker ps`, `docker stop`
   - `docker rm`, `docker rmi`, `docker images`
   - Running commands in containers (`docker exec`)
   - Interactive mode (`-it`) vs detached mode (`-d`)
4. **Working with Containers**
   - Container lifecycle (created, running, paused, stopped, deleted)
   - Inspecting containers (`docker inspect`, `docker logs`)
   - Port publishing (`-p 8080:80`)
   - Environment variables (`-e MY_VAR=value`)
5. **Docker Images & Hub**
   - Docker Hub (official vs. community images)
   - Image tagging (`docker tag`)
   - Pulling and pushing images
6. **Dockerfile Basics**
   - `FROM`, `RUN`, `CMD`, `ENTRYPOINT`
   - `COPY` vs. `ADD`
   - `WORKDIR`, `USER`, `EXPOSE`, `ENV`
   - Building images (`docker build -t myapp .`)
7. **Managing Data**
   - Volumes (`-v /host:/container`)
   - Bind mounts (for development)
   - `tmpfs` mounts
8. **Simple Web App Containerization**
   - Containerizing a static website (Nginx)
   - Containerizing a basic Node.js/Python app

### Hands-On Syllabus: Week 1

**Day 1: Installation & First Container**

```bash
# Verify installation
docker --version
docker run hello-world

# First interactive container
docker run -it ubuntu bash
# Inside container: ls, pwd, echo "Hello", exit

# Run a web server
docker run -d -p 8080:80 nginx
# Open http://localhost:8080
```

_Exercise_: Run an Alpine Linux container, install `curl`, and fetch google.com.

**Day 2: Container Lifecycle**

```bash
# Run, stop, remove
docker run -d --name mynginx nginx
docker ps
docker stop mynginx
docker ps -a
docker rm mynginx

# Container logs
docker run -d --name logger busybox sh -c "while true; do date; sleep 1; done"
docker logs -f logger # Press Ctrl+C to exit
docker stop logger

# Execute commands
docker run -d --name redis redis
docker exec -it redis redis-cli ping
```

_Exercise_: Run a MySQL container, exec into it, and create a database using `mysql -u root -p`.

**Day 3: Working with Images**

```bash
# Image operations
docker pull python:3.9-slim
docker images
docker tag python:3.9-slim mypython:latest
docker rmi mypython:latest

# Image history
docker history nginx:latest

# Save and load images
docker save nginx -o nginx.tar
docker load -i nginx.tar
```

_Exercise_: Pull Ubuntu 20.04, 22.04, and latest. Compare sizes using `docker images`.

**Day 4-5: Dockerfile Basics (Build Your First Images)**

_Project 1: Static Website_

```html
<!-- index.html in same folder -->
<!DOCTYPE html>
<html>
  <body>
    <h1>My Docker Site</h1>
  </body>
</html>
```

```dockerfile
# Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

```bash
docker build -t my-site .
docker run -d -p 8080:80 my-site
```

_Project 2: Python App_

```python
# app.py
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "Hello from Docker!"
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```text
# requirements.txt
flask==2.3.0
```

```dockerfile
# Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

_Exercises_:

- Build and run the Python app on port 5000.
- Modify the code and rebuild (observe caching).
- Create a Dockerfile for a Node.js app.

**Day 6: Volumes & Persistent Data**

```bash
# Named volume
docker volume create mydata
docker run -v mydata:/data alpine touch /data/test.txt
docker run -v mydata:/data alpine ls /data

# Bind mount (for development)
echo "<h1>Live reload</h1>" > index.html
docker run -v $(pwd):/usr/share/nginx/html:ro -p 8080:80 nginx
# Edit index.html and refresh browser - changes appear instantly!

# Backup a volume
docker run --rm -v mydata:/source -v $(pwd):/backup alpine tar czf /backup/mydata-backup.tar.gz /source
```

_Exercise_: Create a PostgreSQL container with a named volume, add some data, delete container, recreate with same volume and verify data persists.

**Day 7: Environment Variables & Port Mapping**

```bash
# Environment variables
docker run -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=mydb -d mysql:8

# Port mapping variations
docker run -p 8080:80 nginx   # host:container
docker run -p 80 nginx        # random host port
docker run -P nginx           # expose all EXPOSE ports

# Inspect configuration
docker inspect mycontainer | grep -A 5 "Env"
docker port mycontainer
```

_Exercise_: Run a WordPress container linked to MySQL using env vars (use `--link` or custom network).

---

## 🟡 Level 2: Intermediate

### Curriculum Topics

9. **Docker Networking**
   - Network drivers: bridge, host, overlay, macvlan, none
   - Creating custom networks (`docker network create`)
   - Connecting containers to networks
   - Container name resolution
   - Exposing vs publishing ports
10. **Docker Compose**
    - `docker-compose.yml` structure (version, services, networks, volumes)
    - Defining multi-container apps (web + db + cache)
    - Commands: `up`, `down`, `logs`, `exec`, `build`
    - Profiles, `depends_on`, healthchecks
11. **Modern Local Development Environments**
    - **Dev Containers (`.devcontainer`)**: VS Code / JetBrains `devcontainer.json`, devcontainer features, mounting source, port forwarding, and workspace UX.
    - **Docker Compose Watch**: Using `docker compose watch` for real-time file sync and automatic container rebuilds during development.
    - **Sync Tools (optional)**: Mutagen, docker-sync for macOS file-performance improvements.
12. **Data Persistence Best Practices**
    - Named volumes vs bind mounts
    - Volume drivers (local, NFS, cloud)
    - Backing up and restoring volumes
13. **Dockerfile Optimization & Advanced Layering Cache**
    - Layer caching and build order
    - Multi-stage builds (reduce image size)
    - `.dockerignore` file
    - Alpine & slim variants
    - Best practices (run as non-root, use `--no-cache` for `apk`)
    - **Language-Specific Caches**: Caching `npm`, `pip`, Go modules, Maven/Gradle layers before copying app source.
14. **Debugging & Troubleshooting**
    - `docker logs --tail`, `--since`
    - Entering broken containers (`docker exec -it sh`)
    - Inspecting resource usage (`docker stats`, `docker system df`)
    - Common errors (port conflicts, volume permission, OOM)
15. **Container Registries & Pull-Through Caching**
    - Private registries (Docker Hub, ECR, GCR, ACR)
    - Authentication (`docker login`)
    - Image naming conventions
    - **Local Registry & Pull-Through Cache**: Running a local registry to avoid rate limits and speed up CI. Basic auth and TLS for local registry.

### Hands-On Syllabus: Week 2

**Day 8-9: Docker Networking**

```bash
# Create custom bridge network
docker network create mynet
docker run -d --name app1 --network mynet nginx
docker run -d --name app2 --network mynet nginx

# Test container communication
docker run --rm --network mynet alpine ping app1
docker run --rm --network mynet alpine wget -O- http://app1

# Network isolation
docker run --rm alpine ping app1 # This will fail (different network)

# Network drivers
docker network create --driver bridge isolated_net
docker network create --driver host host_net # shares host network
```

_Exercise_: Create a three-tier app network (frontend, backend, db) with two custom networks and verify frontend can't ping db directly.

**Day 10-12: Docker Compose (Real Projects)**

_Project 3: LEMP Stack (Linux, Nginx, MySQL, PHP)_

```yaml
# docker-compose.yml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./app:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php

  php:
    image: php:8.2-fpm
    volumes:
      - ./app:/var/www/html

  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

```bash
# Commands
docker-compose up -d
docker-compose ps
docker-compose logs -f nginx
docker-compose exec php bash
docker-compose down -v # remove volumes too
```

_Project 4: Full-Stack App (React + Node + MongoDB)_

```yaml
version: "3.8"

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:5000

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/app
    depends_on:
      - mongo

  mongo:
    image: mongo:6
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

_Exercises_:

- Add a healthcheck to the backend service.
- Add a Redis cache service.
- Use profiles for development vs production.

**Day 13: Dockerfile Optimization**

```dockerfile
# Before (inefficient)
FROM node:18
COPY . .
RUN npm install
RUN npm run build
CMD ["npm", "start"]

# After (multi-stage)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]

# Using build cache efficiently for Python
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt # Cache this layer
COPY . . # Changes frequently, comes last
```

_Exercise_: Optimize a bloated Docker image (start with `ubuntu:latest`, install Python and dependencies, then reduce from 1GB to under 200MB).

**Day 14: Logging & Debugging**

```bash
# Different logging drivers
docker run --log-driver json-file --log-opt max-size=10m nginx
docker run --log-driver none nginx # no logs

# Real-time debugging
docker stats --watch
docker system df -v

# Debugging a failing container
docker run -it --entrypoint /bin/sh myapp # override entrypoint
docker logs --tail 50 mycontainer
docker inspect mycontainer | jq '.[0].State'

# Resource limits in action
docker run --memory=256m --memory-swap=512m --cpus=0.5 stress --vm 1 --vm-bytes 200M
```

_Exercise_: Create a container that fails with OOM killer, inspect the exit code and logs.

---

## 🔴 Level 3: Advanced

### Curriculum Topics

16. **Security Best Practices & Supply Chain**
    - Running as non-root user inside container
    - Seccomp, AppArmor, SELinux profiles
    - Capabilities (drop all, add only required)
    - Image scanning (Trivy, Clair, Docker Scout)
    - Supply chain security (signing with Cosign, SBOMs)
    - **Ephemeral / Debug Containers**: Using `docker debug` or ephemeral sidecar containers to inspect processes without shell access.
    - **Vulnerability Remediation Workflow**: Prioritize CVEs, update base images, pin patched packages, rebuild/redeploy, maintain SBOM.
17. **Resource Management**
    - CPU limits (`--cpus`, `--cpu-shares`)
    - Memory limits (`--memory`, `--memory-swap`)
    - I/O limits (`--blkio-weight`)
    - OOM killer behavior
18. **Custom Docker Networks Deep Dive**
    - User-defined bridge networks (automatic DNS)
    - Macvlan (assign MAC addresses)
    - IPv6 support
    - Network policies (with 3rd-party plugins)
19. **Observability, Logging & Monitoring**
    - Logging drivers (json-file, syslog, fluentd, awslogs, gelf)
    - Centralized logging with ELK/Loki
    - Monitoring with cAdvisor + Prometheus + Grafana
    - Docker health checks (`HEALTHCHECK` instruction)
    - **OpenTelemetry (OTel)**: Instrumenting apps inside containers, running OTel Collector, exporting metrics/traces to backend.
    - **Tracing Patterns**: Correlation IDs and context propagation across microservices.
20. **Docker in Production (The Cloud-Native Reality)**
    - Restart policies (`no`, `on-failure`, `always`, `unless-stopped`)
    - Rolling updates with `docker service` or compose
    - Zero-downtime deployments (blue-green, canary)
    - Secrets management (Docker secrets in Swarm, or HashiCorp Vault)
    - **Where Docker Images Run**: ECS, Fargate, Cloud Run, Azure Container Apps, GKE — patterns and constraints.
    - **Serverless Containers**: Cold-start tradeoffs, stateless container design.
    - **Sidecar & Adapter Patterns**: When to use sidecars (logging, proxy, telemetry, config reloaders).
21. **Orchestration Basics (Prep for Kubernetes)**
    - Docker Swarm (services, stacks, nodes)
    - Differences between Swarm and Kubernetes
    - When to use orchestration

### Hands-On Syllabus: Week 3

**Day 15-16: Security Hardening**

```dockerfile
# Secure Dockerfile example
FROM alpine:3.18

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# Set proper permissions
COPY --chown=appuser:appgroup app /app
RUN chmod 500 /app/main

# Drop all capabilities, add only needed
USER appuser
```

```bash
# Run with security options
docker run --cap-drop=ALL --cap-add=NET_ADMIN --security-opt=no-new-privileges nginx
docker run --read-only --tmpfs /tmp nginx
docker run --security-opt=seccomp=./custom-seccomp.json nginx

# Scan images for vulnerabilities
docker scout cves nginx:latest
trivy image python:3.9-slim
```

_Exercises_:

- Create a Dockerfile that runs a web server as a non-root user on port 8080.
- Use `docker scout` or `trivy` to find vulnerabilities in an old Ubuntu image, then fix them.
- Implement Docker Content Trust: `export DOCKER_CONTENT_TRUST=1`.

**Day 17-18: Production Patterns**

```bash
# Restart policies
docker run --restart=always nginx
docker run --restart=on-failure:5 app-that-might-fail
docker update --restart=unless-stopped mycontainer

# Health checks
docker run --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-retries=3 \
  nginx

# Rolling updates with Compose
docker-compose up -d --no-deps --build web

# Secrets management (Docker Swarm)
echo "dbpassword" | docker secret create db_password -
```

_Project 5: Production-Ready App_

- Implement health checks for each service.
- Add resource limits.
- Set up logging to syslog.
- Create a backup script for volumes.
- Implement zero-downtime deployment.

**Day 19-20: Buildx & Multi-Architecture**

```bash
# Setup buildx
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap

# Build multi-arch image
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myuser/myapp:latest --push .

# Test on different architectures
docker run --platform linux/arm64 myuser/myapp:latest
```

_Exercise_: Build a single image that runs on both Intel and Apple Silicon Macs, push to Docker Hub.

---

## 🔵 Level 4: Expert

### Curriculum Topics

22. **Docker Internals & Linux Kernel**
    - **Namespaces**: `pid`, `net`, `mnt`, `uts`, `ipc`, `user`, `cgroup`. Deep dive with examples of how `pid` and `mount` namespaces isolate processes.
    - **Cgroups v1 vs v2**: Effects on memory accounting, PIDs, and OOM behavior; interaction with `systemd`.
    - Union filesystems (OverlayFS, AUFS)
    - Container runtime: `runc`, `containerd`
    - **eBPF**: Basic concepts and how tools (Cilium, bpftools) use eBPF for high-performance networking and observability.
23. **Custom Docker Extensions & Plugins**
    - Volume, Network, and Authorization plugins
    - BuildKit enhancements
24. **Advanced Build & Reproducibility**
    - `--mount=type=cache`, `--mount=type=secret` (BuildKit)
    - `docker buildx` for multi-architecture builds
    - **Reproducible Builds**: `SOURCE_DATE_EPOCH`, deterministic layer ordering, fixed ownership.
    - **Bake + Buildx**: Orchestrating multi-image parallel builds via bake files.
    - **Remote Cache Backends**: Using `type=gha`, `inline`, or registry caching across CI runners.
25. **CI/CD Pipeline Patterns & Automation**
    - Building & pushing images in GitHub Actions / GitLab CI
    - Caching strategies in CI pipelines (incremental builds for monorepos)
    - Testing Docker images (Container Structure Tests)
    - **SBOM & Supply Chain Automation**: Enforcing policy gates in CI using tools like Syft, automated CVE triage.
    - Security in pipelines: Signed images (Cosign) and verifying provenance.
26. **Container Cost Optimization (FinOps) & Performance**
    - Reducing image size (distroless, scratch images)
    - Faster builds (layer caching, parallel builds)
    - Network and storage driver selection (`overlay2`)
    - **Right-Sizing**: Analyzing historic usage (cAdvisor, Prometheus) to set realistic CPU/memory limits.
    - **Idle Container Management**: Automated stop/sleep for dev/staging, request-driven containers.
    - **Image Footprint**: How image size affects cold-start and storage costs.
27. **Debugging & Forensics**
    - Generating core dumps, using `nsenter` or PID namespace sharing to inspect low-level issues.
    - Forensic steps after container compromise.
    - Real-time daemon events (`docker system events`).
28. **Confidential & Hardware-backed Isolation**
    - gVisor (runsc), Kata Containers
    - Rootless Docker
    - FIPS-compliant images
    - **Confidential Containers (CoCo)**: AMD SEV, Intel TDX/SGX. Intro to use-cases and limitations.
    - **Wasm (WebAssembly) in Docker**: Running Wasm side-by-side or instead of Linux containers for tiny sandboxed workloads.
29. **Migration & Legacy Applications**
    - Dockerizing legacy monoliths, handling stateful applications.
30. **Docker Alternatives & Ecosystem (OCI)**
    - Podman, Buildah, Skopeo: Compatibility patterns, rootless networking caveats.
    - `containerd` directly; Lima (macOS Linux VMs)
    - **OCI & Ecosystem Tooling**: OCI image format, runtimes, CSI basics for container storage.
31. **Real-World Projects & Case Studies**
    - Build a full-stack app with microservices.
    - Migrate a VM-based app to containers.
    - Auto-scaling using Docker + Prometheus + custom scripts.

### Hands-On Syllabus: Week 4

**Day 21: Docker Internals Exploration**

```bash
# Explore namespaces
docker run -d --name nginx nginx
pid=$(docker inspect -f '{{.State.Pid}}' nginx)
sudo ls -la /proc/$pid/ns/

# See cgroups
sudo cat /proc/$pid/cgroup

# Explore overlay filesystem
docker info | grep "Storage Driver"
docker inspect nginx | grep UpperDir

# Use low-level container runtime (runc)
docker run -d --name test alpine sleep 360
sudo runc list
sudo runc exec test ps aux
```

_Exercise_: Manually create a container using `runc` from scratch (without Docker).

**Day 22: Custom Docker Plugins**

```bash
# List plugins
docker plugin ls

# Install a volume plugin (e.g., SSHFS)
docker plugin install vieux/sshfs

# Create a volume with plugin
docker volume create -d vieux/sshfs \
  -o sshcmd=user@host:/remote-dir \
  -o password=secret \
  mysshvolume
```

_Exercise_: Install and use the Netdata monitoring plugin or build a simple authorization plugin.

**Day 23-24: CI/CD Pipeline Project**

_GitHub Actions workflow (`.github/workflows/docker.yml`):_

```yaml
name: Build and Deploy
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Scan for vulnerabilities
        run: trivy image myapp:${{ github.sha }}

      - name: Run tests
        run: docker run myapp:${{ github.sha }} npm test

      - name: Push to registry
        run: |
          docker tag myapp:${{ github.sha }} myrepo/myapp:latest
          docker push myrepo/myapp:latest
```

_Exercise_: Build a complete pipeline that builds an image on PR, runs security scans, pushes to registry on main branch, deploys to staging environment, and runs smoke tests.

**Day 25-26: Performance Tuning Lab**

```bash
# Measure performance
docker run --cpus=2 --memory=1g your-app
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Benchmark different storage drivers
docker run --storage-opt size=10G alpine dd if=/dev/zero of=test bs=1M count=100

# Network performance test
docker run --network=host nginx # vs default bridge
docker run --network=myoverlay nginx

# Use --squash to reduce layers
docker build --squash -t squashed-image .
```

_Project 6_: Benchmark your app with different resource limits and find optimal settings.

**Day 27-28: Real-World Capstone Project**
_Build a Microservices Platform:_

_Services:_

- API Gateway (Traefik/Nginx)
- Authentication Service (Node.js)
- Product Service (Python/FastAPI)
- Order Service (Go)
- Database (PostgreSQL)
- Cache (Redis)
- Message Queue (RabbitMQ)
- Monitoring (Prometheus + Grafana)

_Requirements:_

- Multi-stage builds for each service
- Custom network isolation (frontend/backend/db tiers)
- Health checks and auto-restart
- Log aggregation with ELK or Loki
- Zero-downtime rolling updates
- Secrets management (no env vars for passwords)
- Resource limits for each service
- Backup strategy for databases
- Multi-arch support (amd64 + arm64)
- CI/CD pipeline with automated testing

---

## 📚 Bonus / Cross-Cutting Topics

These topics apply across multiple levels and are essential for mastery:

- **GPU & Hardware Acceleration**: Using the NVIDIA Container Toolkit and `--gpus` flag, device plugin patterns for Kubernetes, and device passthrough constraints.
- **Docker AI/LLM Local Stacks**: Containerizing local AI models using Ollama, LocalAI, and mounting host GPUs.
- **Registry Hardening & DR**: Backup strategies for registries and backing storage, purge policies, immutable tags, retention rules.
- **IPAM & Large-Scale Networking**: IPAM planning: assigning CIDR blocks for container networks, avoiding VPN and corporate subnet conflicts. Network segmentation patterns.
- **Windows Containers**: Fundamentals and differences vs Linux containers, Windows base images (nanoserver, windowsservercore), filesystem/permissions and networking caveats.
- **Docker Desktop Extensions**: Build custom UI plugins.
- **Docker Debug**: Using the experimental `docker debug` command.

---

## 📋 Quick Reference: Commands to Master

| Level            | Must-Know Commands                                                          |
| :--------------- | :-------------------------------------------------------------------------- |
| **Beginner**     | `run`, `ps`, `stop`, `rm`, `images`, `rmi`, `pull`, `exec`, `logs`, `build` |
| **Intermediate** | `network`, `volume`, `compose up/down`, `system df`, `stats`, `inspect`     |
| **Advanced**     | `buildx`, `scout`, `scan`, `swarm`, `stack`, `secret`, `config`             |
| **Expert**       | `plugin`, `system events`, `container cp`, `commit`, `diff`                 |

---

## 🎯 Self-Assessment Checklist

### Beginner ✅

- [ ] Can run any Docker Hub image
- [ ] Can write a Dockerfile for a simple app
- [ ] Understands port mapping and volumes
- [ ] Can debug basic container failures

### Intermediate ✅

- [ ] Uses multi-stage builds effectively
- [ ] Orchestrates 3+ services with Compose
- [ ] Configures custom networks
- [ ] Implements health checks and resource limits

### Advanced ✅

- [ ] Builds secure, non-root containers
- [ ] Uses buildx for multi-arch images
- [ ] Sets up CI/CD pipelines
- [ ] Scans images for vulnerabilities

### Expert ✅

- [ ] Understands namespaces and cgroups
- [ ] Can debug using runc/containerd directly
- [ ] Optimizes image size and build speed
- [ ] Implements production patterns (blue-green, canary)
