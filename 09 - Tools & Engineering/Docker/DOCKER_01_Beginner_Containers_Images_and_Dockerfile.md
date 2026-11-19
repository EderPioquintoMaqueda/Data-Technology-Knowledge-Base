---
tags:
  - docker
  - containerization
  - beginner
  - ibm-infrastructure
  - obsidian-vault
aliases:
  - Docker Beginner Basics
---

# Docker Foundations: Containers, Images, and Dockerfiles

Related notes: [[LIN_01_Command_Line_Navigation_and_File_Management]], [[SQL-B04 Creating Tables - DDL, DML and Constraints]]

---

## 1. What is Docker? (Virtual Machines vs. Containers)

Unlike Virtual Machines (VMs), which virtualize an entire hardware stack including a heavy guest Operating System, **Containers** virtualize only the OS kernel layer. They share the host machine's kernel, making them lightweight, blazing fast, and consuming minimal RAM.

---

## 2. Core Concepts: Images vs. Containers

* **Docker Image:** A read-only template containing application code, libraries, dependencies, and environment settings (comparable to a class in object-oriented programming).
* **Docker Container:** A running, instanced execution of a Docker image (comparable to an object instance).

### Essential CLI Commands Reference
```bash
# 1. Check Docker version and daemon status
docker --version

# 2. List all running containers
docker ps

# 3. List all containers (including stopped ones)
docker ps -a

# 4. List all downloaded local images
docker images

# 5. Stop a running container gracefully
docker stop <container_id>

# 6. Remove a stopped container
docker rm <container_id>
```

---

## 3. Running Your First Containers
```bash
docker run hello-world
docker run -it --rm python:3.12-slim python        # interactive Python, removed on exit
docker run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 postgres:17
#          │   │          │                          └ host:container port
#          │   │          └ environment variable
#          │   └ container name
#          └ detached (background)
docker logs -f pg
docker exec -it pg psql -U postgres                # open psql INSIDE the container
docker stop pg && docker start pg
```

## 4. Your First Dockerfile
```dockerfile
FROM python:3.12-slim                 # base image
WORKDIR /app                          # working directory inside the image
COPY requirements.txt .               # copy dependencies first (better layer caching)
RUN pip install --no-cache-dir -r requirements.txt
COPY . .                              # then the code
CMD ["python", "main.py"]             # default command
```
```bash
docker build -t sales-etl:0.1 .
docker run --rm --env-file .env sales-etl:0.1
```
Each instruction creates a cached **layer**; put things that change rarely (dependencies) **before** things that change often (code).

### `.dockerignore`
```
.venv
.git
__pycache__
data/
.env
```

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| container | /kənˈteɪ.nɚ/ | kon-**TÉI**-ner | contenedor |
| image | /ˈɪm.ɪdʒ/ | **I**-mich | imagen |
| layer | /ˈleɪ.ɚ/ | **LÉI**-er | capa |
| build | /bɪld/ | bild | construir |
| detached | /dɪˈtætʃt/ | di-**TACHT** | separado (en segundo plano) |
| lightweight | /ˈlaɪt.weɪt/ | **LÁIT**-ueit | ligero |
| kernel | /ˈkɝː.nəl/ | **KER**-nel | núcleo |

➡️ [[DOCKER_02_Intermediate_Volumes_Networks_and_Compose]] · 🏠 [[00 - Tools Roadmap]]
