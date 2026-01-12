---
tags: ['docker', 'containers', 'docker-compose', 'devops', 'tools', 'roadmap']
---

# Docker Compose

## Summary

Docker Compose is a tool for defining and running multi-container Docker applications using YAML configuration files. It simplifies development workflows by managing entire application stacks (web server, database, cache, etc.) with a single command. Compose handles networking, volume creation, and service dependencies automatically, making it ideal for local development, testing, and single-host deployments.

## Detailed Explanation

### What is Docker Compose

```yaml
# Docker Compose is NOT Docker
# It's a tool that USES Docker

# Composition workflow:
# 1. Define services in docker-compose.yml
# 2. Run: docker compose up
# 3. Compose creates network, volumes, starts all containers

# Before Compose (manual):
# docker run -d -v postgres_data:/var/lib/postgresql/data postgres:15
# docker run -d -v redis_data:/data redis:7-alpine
# docker run -d -v app_data:/app/data --network bridge myapp:latest
# (Hard to manage, manual wiring)

# With Compose (automatic):
# docker compose up
# (Everything configured in YAML, wired together)
```

### Docker Compose Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Compose CLI                      │
│  ┌──────────────────────────────────────────────────┐    │
│  │  docker-compose.yml / docker-compose.yaml   │    │
│  └──────────────────────────────────────────────────┘    │
│                         │                          │       │
│                         │ PARSING                   │       │
│                         ▼                          │       │
│  ┌──────────────────────────────────────────────────┐    │
│  │  Service Definitions                        │    │
│  │  Services:                                  │    │
│  │    - App (web)                              │    │
│  │    - Database (postgres)                        │    │
│  │    - Cache (redis)                            │    │
│  │    - Queue (rabbitmq)                        │    │
│  └──────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    Docker Engine (creates containers, networks, volumes)
```

### Basic docker-compose.yml Structure

```yaml
version: '3.8'  # Compose file format version

services:
  # Service name (unique)
  web:
    # Image or build
    image: nginx:alpine
    
    # Container name (optional)
    container_name: myapp-web
    
    # Restart policy
    restart: unless-stopped
    
    # Port mapping
    ports:
      - "8080:80"
    
    # Environment variables
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://postgres:5432/app
    
    # Environment from file
    env_file:
      - .env.production
    
    # Volumes
    volumes:
      - app_data:/app/data
      - ./config:/app/config:ro
    
    # Networks
    networks:
      - app-network
    
    # Dependencies
    depends_on:
      postgres:
        condition: service_healthy
      redis:
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  # Named volumes
  app_data:
    driver: local

networks:
  # Custom networks
  app-network:
    driver: bridge
```

### Build vs Image Services

```yaml
# IMAGE SERVICE (from registry)
services:
  web:
    image: nginx:alpine
    # Simple, fast, no build required
    # Use when image exists in registry

# BUILD SERVICE (from Dockerfile)
services:
  app:
    build: .  # Path to Dockerfile
    # Can specify Dockerfile path
    build:
      context: ./app
      dockerfile: Dockerfile.prod
    # Build arguments
    args:
      - BUILD_VERSION=production
    # Build with cache
    cache_from:
      - myregistry/cache:latest
```

### Networks and Communication

```yaml
# Default: All services on same network
services:
  web:
    image: nginx
    # Can access other services by service name
    environment:
      - API_URL=http://api:8000
  
  api:
    image: myapp:latest
    # Can access web by service name
    environment:
      - WEB_URL=http://web:80

# Custom networks
services:
  frontend:
    networks:
      - frontend-net
  
  backend:
    networks:
      - frontend-net
      - backend-net
  
  database:
    networks:
      - backend-net

networks:
  frontend-net:
  backend-net:
```

### Volumes in Compose

```yaml
# NAMED VOLUMES (managed by Compose)
volumes:
  db_data:        # Compose creates automatically
  cache_data:
  uploads_data:

# Using volumes in services
services:
  postgres:
    image: postgres:15
    volumes:
      - db_data:/var/lib/postgresql/data  # Named volume
      - ./init:/docker-entrypoint-initdb.d:ro  # Bind mount

# EXTERNAL VOLUMES (created outside Compose)
volumes:
  persistent_data:
    external: true  # Must exist before compose up
```

### Environment Variables

```yaml
# INLINE ENVIRONMENT
services:
  app:
    environment:
      - DATABASE_URL=postgres://localhost/db
      - REDIS_HOST=redis
      - LOG_LEVEL=info

# FROM FILE (.env)
services:
  app:
    env_file:
      - .env

# FROM MULTIPLE FILES
services:
  app:
    env_file:
      - .env.common
      - .env.production

# VALUE FROM HOST ENVIRONMENT
services:
  app:
    environment:
      - HOSTNAME=${HOSTNAME:-localhost}
      - PORT=${PORT:-3000}
```

```bash
# .env file example
DATABASE_URL=postgres://postgres:5432/app
REDIS_HOST=redis
LOG_LEVEL=debug
SECRET_KEY=your-secret-key-here

# Run with environment file
docker compose --env-file .env.production up
```

### Profiles

```yaml
# PROFILES - Run different sets of services
services:
  app:
    image: myapp:latest
    profiles:
      - development  # Only with --profile development
    environment:
      - NODE_ENV=development
      - DEBUG=true
  
  app-prod:
    image: myapp:latest
    profiles:
      - production  # Only with --profile production
    environment:
      - NODE_ENV=production
      - DEBUG=false

  # No profile = always runs
  monitoring:
    image: prometheus
```

```bash
# Run specific profiles
docker compose --profile development up
docker compose --profile production up

# Multiple profiles
docker compose --profile development --profile monitoring up
```

### Dependencies and Startup Order

```yaml
# SIMPLE DEPENDENCY (waits for container start)
services:
  web:
    depends_on:
      - db
      - cache
    image: nginx

# CONDITIONAL DEPENDENCY (waits for health)
services:
  web:
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    image: nginx

# LONG SYNTAX (same)
services:
  web:
    depends_on:
      - db
      - cache

# NESTED CONDITIONS
services:
  web:
    depends_on:
      db:
        condition: service_healthy
    image: nginx
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
```

### Compose Commands

```bash
# START SERVICES
docker compose up                    # Create and start all services
docker compose up -d                # Detached mode (background)
docker compose up --build             # Rebuild before starting
docker compose up --force-recreate     # Recreate containers
docker compose up --no-deps           # Don't start dependencies

# STOP SERVICES
docker compose stop                   # Stop all services
docker compose stop web               # Stop specific service
docker compose down                    # Stop and remove containers, networks
docker compose down -v                # Also remove volumes (DANGEROUS!)

# STATUS AND LOGS
docker compose ps                     # List running services
docker compose logs                   # Show all logs
docker compose logs -f web            # Follow specific service logs
docker compose logs --tail=100 web   # Last 100 lines

# EXECUTE COMMANDS
docker compose exec web bash           # Run command in service
docker compose run web python script.py  # Run one-off command
docker compose run --rm web python script.py  # Remove after

# BUILD AND PUSH
docker compose build                 # Build all services
docker compose build web             # Build specific service
docker compose push                   # Push all services to registry

# CONFIGURATION
docker compose config                  # Validate and show final configuration
docker compose config --services        # Show only services
docker compose -f docker-compose.prod.yml up  # Use different compose file
```

### Multi-File Projects

```yaml
# MAIN docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:15
    env_file:
      - .env

# INCLUDED FILE (docker-compose.override.yml)
services:
  app:
    environment:
      - DEBUG=true  # Override for local development

# COMPOSE USES BOTH FILES
# docker compose up
# Merge strategy: main + override
```

### Development Workflow

```yaml
# COMPLETE DEVELOPMENT STACK
version: '3.8'

services:
  # Frontend (React/Vue/Angular)
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules  # Prevent host node_modules
    environment:
      - CHOKIDAR_USE_POLLING=true
    command: npm start

  # Backend API
  api:
    build: ./api
    ports:
      - "8000:8000"
    volumes:
      - ./api:/app
    environment:
      - DATABASE_URL=postgres://postgres:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
    command: uvicorn app:main --reload

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  # Admin Tools
  pgadmin:
    image: dpageadmin/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_SERVER: postgres
      PGADMIN_SERVER_PORT: 5432
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
```

```bash
# Development workflow
docker compose up -d                # Start all services
docker compose logs -f api            # Watch API logs
docker compose exec api pytest       # Run tests
docker compose down                   # Stop everything when done
```

### Production Considerations

```yaml
# PRODUCTION DOCKER-COMPOSE.YML

version: '3.8'

services:
  app:
    image: mycompany/myapp:v1.2.3  # Specific version, not latest
    restart: always
    environment:
      - NODE_ENV=production
    env_file:
      - .env.production  # Never commit this file
    volumes:
      - app_data:/app/data  # Named volume, not bind mount
      - ./certs:/app/certs:ro  # Read-only config
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      replicas: 3  # For Docker Swarm
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - nginx_data:/var/log/nginx
    depends_on:
      app:
        condition: service_healthy
```

### Advanced Patterns

```yaml
# YAML ANCHORS AND EXTENSIONS
x-shared-env: &shared-env
  RACK_ENV: production
  RACK_TIMEOUT: 30

x-db-connection: &db-connection
  POSTGRES_HOST: postgres
  POSTGRES_PORT: 5432

services:
  web:
    environment:
      <<: *shared-env
      <<: *db-connection

  worker:
    environment:
      <<: *shared-env
      <<: *db-connection

# EXTENSION FILES (docker-compose.*.yml)
# Automatically merged by Compose
# docker-compose.override.yml (local overrides)
# docker-compose.prod.yml (production)
# docker-compose.dev.yml (development)
```

```yaml
# SERVICES WITHOUT NETWORKING (for external network)
services:
  app:
    network_mode: host  # Use host network
    image: myapp

# EXTERNAL SERVICES (already running)
services:
  app:
    image: myapp
    external_links:
      - database  # Existing container named "database"
```

## Interview Questions

### Q1: What is Docker Compose?
**A:** Docker Compose is a tool for defining and running multi-container Docker applications using YAML files. It manages service dependencies, networking, and volumes automatically with a single `docker compose up` command.

### Q2: How do you specify build vs image in Compose?
**A:** Use `image: nginx` for pre-built images from registries. Use `build: .` or `build: {context: ./app, dockerfile: Dockerfile.prod}` to build from a Dockerfile. Build context and Dockerfile path can be customized.

### Q3: What is the purpose of profiles in Docker Compose?
**A:** Profiles allow running different subsets of services (e.g., `docker compose --profile development up`). This enables separating development, testing, and production configurations in the same compose file, or running optional services like monitoring.

### Q4: How do you handle dependencies between services?
**A:** Use `depends_on` with optional conditions. Simple dependency: `depends_on: - db` waits for db container to start. Conditional: `depends_on: {db: {condition: service_healthy}}` waits for db's health check to pass.

### Q5: What is the difference between `docker compose down` and `docker compose down -v`?
**A:** `docker compose down` stops and removes containers and networks. The `-v` flag also removes volumes, which destroys all data (dangerous!). Never use `-v` in production without understanding the consequences.

### Q6: How do you use environment files with Compose?
**A:** Use `env_file: - .env` to load environment variables from a file. Multiple files can be specified with `env_file: - .env.common - .env.production`. Variables in files override those inline.

### Q7: How do you override configurations for different environments?
**A:** Use multiple compose files with `docker compose -f docker-compose.yml -f docker-compose.override.yml up`. The override file merges with and overrides the base configuration. Create files for different environments (dev, staging, prod).

### Q8: What is the purpose of named volumes in Compose?
**A:** Named volumes in the `volumes:` section are created and managed by Compose. They provide persistent storage that survives container removal, can be shared between services, and are portable across different Compose projects.
