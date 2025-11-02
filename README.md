# Production Docker Server Configuration

> A production-grade, containerized infrastructure setup for running WAHA (WhatsApp HTTP API) with PostgreSQL (pgvector) and Redis on a personal Linux server.

[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-336791?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-Latest-DC382D?logo=redis&logoColor=white)](https://redis.io/)

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Service Details](#service-details)
- [Health Monitoring](#health-monitoring)
- [Backup & Recovery](#backup--recovery)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Performance Tuning](#performance-tuning)

## 🏗️ Architecture Overview

This setup implements a **three-tier containerized architecture** with proper service isolation, health checks, and persistent storage:

```
┌─────────────────────────────────────────────────────┐
│                    Host System                       │
│  ┌───────────────────────────────────────────────┐  │
│  │          Docker Network (Bridge)              │  │
│  │                                               │  │
│  │  ┌─────────────┐     ┌──────────────┐       │  │
│  │  │    WAHA     │────▶│    Redis     │       │  │
│  │  │  Messenger  │     │   (Cache)    │       │  │
│  │  │   :3000     │     └──────────────┘       │  │
│  │  └─────────────┘              │              │  │
│  │         │                     │              │  │
│  │         │                     │              │  │
│  │         ▼                     ▼              │  │
│  │  ┌─────────────────────────────────┐        │  │
│  │  │        PostgreSQL 17            │        │  │
│  │  │      (with pgvector)            │        │  │
│  │  │         :5432                   │        │  │
│  │  └─────────────────────────────────┘        │  │
│  │                                               │  │
│  └───────────────────────────────────────────────┘  │
│                                                      │
│  Volumes: postgres_data, sessions, media            │
└─────────────────────────────────────────────────────┘
```

### Design Principles Applied

- **Separation of Concerns**: Each service runs in isolated containers
- **Dependency Inversion**: Services depend on abstractions (Docker networks) not concrete implementations
- **Health-First Design**: Comprehensive health checks ensure service reliability
- **Resource Constraints**: CPU and memory limits prevent resource exhaustion
- **Graceful Degradation**: Restart policies ensure automatic recovery

## 🛠️ Tech Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Messaging API** | WAHA (WhatsApp HTTP API) | Latest | WhatsApp Business API interface |
| **Database** | PostgreSQL + pgvector | 17 | Relational database with vector similarity search |
| **Cache/Queue** | Redis | Latest | Session management and message queuing |
| **Orchestration** | Docker Compose | v2+ | Container orchestration |

## ✨ Features

### Production-Ready Infrastructure

- ✅ **Health Checks**: All services include comprehensive health monitoring
- ✅ **Auto-Restart**: Services automatically restart on failure
- ✅ **Dependency Management**: Proper startup order with health condition checks
- ✅ **Resource Limits**: PostgreSQL constrained to 1 CPU core and 1GB RAM
- ✅ **Persistent Storage**: Named volumes for data persistence
- ✅ **Vector Search**: pgvector extension for AI/ML embeddings support

### DevOps Best Practices

- 📦 **Containerization**: Fully containerized stack
- 🔒 **Environment Isolation**: Configuration via environment variables
- 📊 **Observability**: Built-in health endpoints
- 🔄 **Stateless Design**: Application state stored in dedicated services
- 💾 **Data Persistence**: Separate volumes for database, sessions, and media

## 📦 Prerequisites

- **Linux Server** (Ubuntu 20.04+ / Debian 11+ / RHEL 8+)
- **Docker Engine** 20.10+
- **Docker Compose** v2.0+
- **Minimum Resources**:
  - 2 CPU cores
  - 4GB RAM
  - 20GB disk space

### Installation of Prerequisites

```bash
# Install Docker (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose (if not included)
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/devherik/PrivateServerExmaple.git
cd PrivateServerExmaple
```

### 2. Create Environment File

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```bash
nano .env
```

### 3. Create Required Directories

```bash
mkdir -p sessions media
chmod 755 sessions media
```

### 4. Start the Services

```bash
# Start in detached mode
docker compose up -d

# View logs
docker compose logs -f

# Check service status
docker compose ps
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the project root with the following variables:

```env
# PostgreSQL Configuration
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=waha_db

# WAHA Configuration (add your specific variables)
# Refer to: https://waha.devlike.pro/docs/overview/environment
WHATSAPP_API_KEY=your_api_key
WHATSAPP_WEBHOOK_URL=https://your-webhook-url.com

# Optional: Redis Configuration
# REDIS_PASSWORD=your_redis_password
```

### Security Recommendations

1. **Strong Passwords**: Use passwords with 16+ characters, including special characters
2. **Secret Management**: Consider using Docker secrets or external secret managers
3. **Network Isolation**: Keep database internal (don't expose port 5432 publicly)
4. **Regular Updates**: Keep all images updated for security patches

```bash
# Update all services
docker compose pull
docker compose up -d
```

## 📖 Usage

### Starting the Stack

```bash
# Start all services
docker compose up -d

# Start specific service
docker compose up -d waha
```

### Stopping the Stack

```bash
# Stop all services
docker compose down

# Stop and remove volumes (⚠️ deletes data)
docker compose down -v
```

### Viewing Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f waha

# Last 100 lines
docker compose logs --tail=100 waha
```

### Accessing Services

- **WAHA API**: http://localhost:3000
- **PostgreSQL**: localhost:5432 (⚠️ exposed for development - secure in production)
- **Redis**: Internal only (no external access)

## 🔍 Service Details

### WAHA (WhatsApp HTTP API)

- **Image**: `devlikeapro/waha:latest`
- **Port**: 3000
- **Health Check**: HTTP GET /health every 30s
- **Volumes**:
  - `./sessions`: WhatsApp session storage
  - `./media`: Media files (images, videos, documents)

### Redis

- **Image**: `redis:latest`
- **Purpose**: Session management, message queue, caching
- **Health Check**: `redis-cli ping` every 10s
- **Network**: Internal only (not exposed)

### PostgreSQL + pgvector

- **Image**: `pgvector/pgvector:pg17`
- **Port**: 5432
- **Features**:
  - pgvector extension for vector similarity search
  - Ideal for RAG (Retrieval-Augmented Generation) applications
  - Embedding storage for AI/ML workloads
- **Resource Limits**:
  - CPU: 1 core
  - Memory: 1GB limit, 256MB reservation
- **Volume**: `postgres_data` (persistent)

## 🏥 Health Monitoring

All services implement health checks for monitoring and automatic recovery:

### Check Service Health

```bash
# All services status
docker compose ps

# Detailed health information
docker inspect waha_messenger | jq '.[0].State.Health'

# Watch health status in real-time
watch -n 2 'docker compose ps'
```

### Health Check Endpoints

```bash
# WAHA health
curl http://localhost:3000/health

# PostgreSQL health
docker compose exec postgres pg_isready -U $POSTGRES_USER

# Redis health
docker compose exec redis redis-cli ping
```

## 💾 Backup & Recovery

### Database Backup

```bash
# Backup PostgreSQL
docker compose exec postgres pg_dump -U $POSTGRES_USER $POSTGRES_DB > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
docker compose exec -T postgres psql -U $POSTGRES_USER $POSTGRES_DB < backup_20250102_120000.sql
```

### Volume Backup

```bash
# Backup volumes
docker run --rm -v dockerserver_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz /data

# Restore volumes
docker run --rm -v dockerserver_postgres_data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/postgres_backup_20250102.tar.gz -C /
```

### Full System Backup

```bash
#!/bin/bash
# backup.sh - Complete backup script

BACKUP_DIR="./backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup database
docker compose exec postgres pg_dump -U $POSTGRES_USER $POSTGRES_DB > $BACKUP_DIR/database.sql

# Backup volumes
cp -r ./sessions $BACKUP_DIR/
cp -r ./media $BACKUP_DIR/

# Backup configuration
cp .env $BACKUP_DIR/
cp docker-compose.yml $BACKUP_DIR/

echo "Backup completed: $BACKUP_DIR"
```

## 🔧 Troubleshooting

### Common Issues

#### Services Won't Start

```bash
# Check logs for errors
docker compose logs

# Verify environment variables
docker compose config

# Check port conflicts
sudo netstat -tulpn | grep -E '3000|5432'
```

#### Database Connection Issues

```bash
# Test PostgreSQL connection
docker compose exec postgres psql -U $POSTGRES_USER -d $POSTGRES_DB -c "SELECT version();"

# Check if database is ready
docker compose exec postgres pg_isready -U $POSTGRES_USER
```

#### Redis Connection Issues

```bash
# Test Redis connection
docker compose exec redis redis-cli ping

# Check Redis logs
docker compose logs redis
```

#### Container Keeps Restarting

```bash
# View container restart count
docker compose ps

# Check health check failures
docker inspect waha_messenger | jq '.[0].State.Health.Log'

# View resource usage
docker stats
```

### Performance Issues

```bash
# Monitor resource usage
docker stats

# Check disk space
df -h

# Optimize PostgreSQL (run inside container)
docker compose exec postgres psql -U $POSTGRES_USER -d $POSTGRES_DB -c "VACUUM ANALYZE;"
```

## 🔒 Security Considerations

### Production Hardening Checklist

- [ ] Change all default passwords
- [ ] Remove PostgreSQL port exposure (5432) if not needed externally
- [ ] Implement firewall rules (UFW/iptables)
- [ ] Use Docker secrets instead of `.env` files
- [ ] Enable TLS/SSL for PostgreSQL connections
- [ ] Implement Redis password authentication
- [ ] Regular security updates (`docker compose pull`)
- [ ] Setup automated backups
- [ ] Implement log rotation
- [ ] Use reverse proxy (Nginx/Traefik) with SSL for WAHA
- [ ] Enable Docker Content Trust

### Firewall Configuration (UFW)

```bash
# Allow SSH
sudo ufw allow 22/tcp

# Allow WAHA (if public)
sudo ufw allow 3000/tcp

# Block PostgreSQL from external access
sudo ufw deny 5432/tcp

# Enable firewall
sudo ufw enable
```

## ⚡ Performance Tuning

### PostgreSQL Optimization

Add to `docker-compose.yml` under postgres `environment`:

```yaml
environment:
  - POSTGRES_USER=${POSTGRES_USER}
  - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
  - POSTGRES_DB=${POSTGRES_DB}
  # Performance tuning
  - POSTGRES_SHARED_BUFFERS=256MB
  - POSTGRES_WORK_MEM=16MB
  - POSTGRES_MAINTENANCE_WORK_MEM=128MB
  - POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
```

### Redis Optimization

```yaml
redis:
  command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
```

## 📊 Monitoring (Optional)

### Add Prometheus + Grafana

```yaml
# Add to docker-compose.yml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
  ports:
    - "9090:9090"

grafana:
  image: grafana/grafana:latest
  ports:
    - "3001:3000"
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=admin
```

## 🤝 Contributing

This is a personal infrastructure project, but suggestions are welcome:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with improvements

## 📝 License

MIT License - Feel free to use this configuration for your own projects.

## 👤 Author

**Herik Colares**
- GitHub: [@devherik](https://github.com/devherik)

## 🙏 Acknowledgments

- [WAHA Project](https://waha.devlike.pro/) - WhatsApp HTTP API
- [pgvector](https://github.com/pgvector/pgvector) - PostgreSQL vector extension
- Docker Community

---

**⭐ If this project helped you, consider giving it a star!**
