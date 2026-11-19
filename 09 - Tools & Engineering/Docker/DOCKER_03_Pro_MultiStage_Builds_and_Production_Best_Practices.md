---
tags:
  - docker
  - containerization
  - pro
  - production
  - obsidian-vault
aliases:
  - Docker Multi-Stage Builds and Security
---

# Docker Pro: Multi-Stage Builds and Production Optimization

Related notes: [[DOCKER_02_Intermediate_Volumes_Networks_and_Compose]], [[LIN_02_Permissions_Processes_and_System_Monitoring]]

---

## 1. The Challenge of Image Bloat

When building Python data science or machine learning containers, installing heavy compilation tools (like `gcc`, `build-essential`, or `pandas`/`scikit-learn` C extensions) bloats the final Docker image size to over 1GB. Larger images increase security vulnerabilities, slow down CI/CD push/pull speeds, and consume unnecessary cloud storage.

---

## 2. Multi-Stage Docker Builds

**Multi-stage builds** allow you to use multiple `FROM` instructions in a single Dockerfile. Each `FROM` instruction starts a *new stage* with a fresh base image. You can selectively copy compiled artifacts and installed Python packages from the heavy build stage into a lean, lightweight production runtime image, discarding compilers and cache files entirely.

### Production-Grade Multi-Stage Dockerfile for Python

```dockerfile
# ==========================================
# STAGE 1: Build & Compile Dependencies
# ==========================================
FROM python:3.10-slim AS builder

WORKDIR /app

# Install system build dependencies required for compiling scientific packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies into a local user directory
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Copy application source code
COPY . /app


# ==========================================
# STAGE 2: Lean Production Runtime
# ==========================================
FROM python:3.10-slim AS runner

WORKDIR /app

# Copy only the installed python packages from the builder stage
COPY --from=builder /root/.local /root/.local
COPY --from=builder /app /app

# Ensure user site-packages are in PATH
ENV PATH=/root/.local/bin:$PATH

# Run container as a non-root user for enterprise security compliance
RUN useradd -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Default execution command
CMD ["python3", "main_pipeline.py"]
```

> ⚠️ **Correction to the example above:** packages installed with `pip install --user` as root live in `/root/.local`, which the non-root `appuser` cannot read (`/root` is mode 700). Use a virtual environment that is owned by the app user, as below.

### Improved multi-stage Dockerfile (2026 style, with uv)
```dockerfile
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project      # deps only → cached layer
COPY src ./src
RUN uv sync --frozen --no-dev

FROM python:3.12-slim AS runtime
RUN useradd --create-home --uid 1000 appuser
WORKDIR /app
COPY --from=builder --chown=appuser:appuser /app /app
ENV PATH="/app/.venv/bin:$PATH" PYTHONUNBUFFERED=1
USER appuser
HEALTHCHECK CMD python -c "import sys; sys.exit(0)"
ENTRYPOINT ["python", "-m", "sales_pipeline.cli"]
```

---

## 3. Production Checklist
| Practice | Why |
|---|---|
| Pin base image versions (`python:3.12.7-slim`) or digests | reproducible builds |
| Small base images (`-slim`, distroless) | fewer vulnerabilities, faster pulls |
| Non-root `USER` | security |
| No secrets in images (`ENV PASSWORD=` ❌) | use env vars / secret managers at runtime |
| `.dockerignore` | smaller context, no leaked files |
| One process per container | simpler scaling & logs |
| Log to stdout/stderr | collected by Docker/Kubernetes |
| `HEALTHCHECK` | orchestrators restart unhealthy containers |
| Scan images (`docker scout cves`, Trivy) | find known CVEs |
| Resource limits (`--memory 2g --cpus 2`) | prevent noisy neighbours |

## 4. Debugging
```bash
docker history sales-etl:0.1          # layer sizes
docker run -it --entrypoint bash sales-etl:0.1
docker inspect <container>
docker stats                           # live CPU/RAM
docker system df && docker system prune   # clean disk (careful!)
```

## 5. Beyond Docker
- **Registries:** Docker Hub, GitHub Container Registry, AWS ECR, IBM Cloud Container Registry.
- **Orchestration:** Kubernetes (pods, deployments, services), managed: EKS, GKE, AKS, OpenShift.
- **CI/CD:** build and push images automatically on every merge (GitHub Actions).

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| multi-stage | /ˌmʌl.ti ˈsteɪdʒ/ | mal-ti **STÉICH** | multietapa |
| bloat | /bloʊt/ | blout | inflar (tamaño excesivo) |
| vulnerability | /ˌvʌl.nɚ.əˈbɪl.ə.t̬i/ | val-ne-ra-**BI**-li-ti | vulnerabilidad |
| registry | /ˈredʒ.ə.stri/ | **RE**-yis-tri | registro |
| deploy | /dɪˈplɔɪ/ | di-**PLOI** | desplegar |
| runtime | /ˈrʌn.taɪm/ | **RAN**-taim | tiempo de ejecución |
| hardening | /ˈhɑːr.dən.ɪŋ/ | **JAR**-de-ning | endurecimiento (seguridad) |

⬅️ [[DOCKER_02_Intermediate_Volumes_Networks_and_Compose]] · Deep dives (ES): [[Docker]] · [[Docker_for_DS]] · 🏠 [[00 - Tools Roadmap]]
