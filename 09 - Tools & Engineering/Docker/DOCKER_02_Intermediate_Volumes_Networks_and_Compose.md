---
tags:
  - docker
  - docker-compose
  - intermediate
  - databases
  - obsidian-vault
aliases:
  - Docker Compose and Volumes
---

# Docker Intermediate: Volumes, Networks, and Docker Compose

Related notes: [[DOCKER_01_Beginner_Containers_Images_and_Dockerfile]], [[SQL-B04 Creating Tables - DDL, DML and Constraints]]

---

## 1. Persistent Storage with Docker Volumes

By default, when a Docker container is deleted, all data created inside its filesystem is permanently lost. To persist data (such as PostgreSQL database files or generated CSV datasets), we use **Docker Volumes**.

### Essential Volume Commands
```bash
# 1. Create a named Docker volume for database persistence
docker volume create postgres_data_vol

# 2. Run a PostgreSQL container attached to the volume
docker run --name pg_container \
  -e POSTGRES_PASSWORD=secret \
  -v postgres_data_vol:/var/lib/postgresql/data \
  -p 5432:5432 \
  -d postgres:13
```

Prefer a current major version (`postgres:17`) for new projects. Bind mounts map a host folder instead: `-v ./sql:/docker-entrypoint-initdb.d` (scripts there run on first start — perfect to load [[SQL Practice Database]]).

---

## 2. Networks
Containers on the same user-defined network reach each other **by name**:
```bash
docker network create data_net
docker run -d --name db  --network data_net -e POSTGRES_PASSWORD=secret postgres:17
docker run --rm --network data_net -e DB_HOST=db sales-etl:0.1     # connects to host "db"
```

## 3. Docker Compose — a whole stack in one file
```yaml
# compose.yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_USER: shop
      POSTGRES_PASSWORD: ${DB_PASSWORD}          # read from .env
      POSTGRES_DB: datashop
    ports: ["5432:5432"]
    volumes:
      - pg_data:/var/lib/postgresql/data
      - ./sql:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U shop -d datashop"]
      interval: 5s
      retries: 10

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}
    ports: ["8080:80"]
    depends_on: [db]

  etl:
    build: .
    environment:
      DB_URL: postgresql+psycopg://shop:${DB_PASSWORD}@db:5432/datashop
    depends_on:
      db:
        condition: service_healthy

volumes:
  pg_data:
```
```bash
docker compose up -d          # start everything
docker compose ps
docker compose logs -f etl
docker compose run --rm etl python main.py --date 2026-11-18
docker compose down           # stop (add -v to delete volumes ⚠️)
```

## 🧪 Practice
1. Start PostgreSQL + pgAdmin with Compose and load the DataShop script automatically.
2. Connect from DBeaver on your machine to `localhost:5432`.
3. Delete the container (not the volume) and prove the data survived.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| volume | /ˈvɑːl.juːm/ | **VAL**-yum | volumen |
| persist | /pɚˈsɪst/ | per-**SIST** | persistir |
| mount | /maʊnt/ | maunt | montar |
| network | /ˈnet.wɝːk/ | **NET**-uerk | red |
| health check | /helθ tʃek/ | jelz chek | verificación de salud |
| depends on | /dɪˈpendz ɑːn/ | di-**PENDZ** an | depende de |

⬅️ [[DOCKER_01_Beginner_Containers_Images_and_Dockerfile]] · ➡️ [[DOCKER_03_Pro_MultiStage_Builds_and_Production_Best_Practices]]
