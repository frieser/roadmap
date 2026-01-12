---
tags: ['docker', 'containers', 'databases', 'devops', 'tools', 'roadmap']
---

# Running Databases in Docker

## Summary

Running databases in Docker containers is ideal for development, testing, and CI/CD environments. Docker provides quick setup, consistent environments, and easy cleanup. For production, additional considerations around data persistence, performance, and backups are required. This guide covers running popular databases (PostgreSQL, MySQL, MongoDB, Redis) in containers with proper configuration.

## Detailed Explanation

### PostgreSQL

```bash
# Basic PostgreSQL
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  postgres:15

# Production-ready configuration
docker run -d \
  --name postgres \
  -e POSTGRES_USER=myapp \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_DB=myappdb \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -v postgres_data:/var/lib/postgresql/data \
  -v ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro \
  -p 5432:5432 \
  --restart unless-stopped \
  postgres:15-alpine

# Environment variables:
# POSTGRES_PASSWORD  - Required (superuser password)
# POSTGRES_USER      - Create this user (default: postgres)
# POSTGRES_DB        - Create this database (default: POSTGRES_USER)
# PGDATA            - Data directory path

# Connect with psql
docker exec -it postgres psql -U myapp -d myappdb

# Or from host (if psql installed)
psql -h localhost -U myapp -d myappdb

# Backup
docker exec postgres pg_dump -U myapp myappdb > backup.sql

# Restore
docker exec -i postgres psql -U myapp myappdb < backup.sql
```

```yaml
# docker-compose.yml for PostgreSQL
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    environment:
      POSTGRES_USER: myapp
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: myappdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp -d myappdb"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### MySQL / MariaDB

```bash
# Basic MySQL
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -p 3306:3306 \
  mysql:8

# Production configuration
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=myappdb \
  -e MYSQL_USER=myapp \
  -e MYSQL_PASSWORD=mypassword \
  -v mysql_data:/var/lib/mysql \
  -v ./my.cnf:/etc/mysql/conf.d/my.cnf:ro \
  -v ./init:/docker-entrypoint-initdb.d:ro \
  -p 3306:3306 \
  --restart unless-stopped \
  mysql:8

# MariaDB (MySQL alternative)
docker run -d \
  --name mariadb \
  -e MARIADB_ROOT_PASSWORD=rootpassword \
  -e MARIADB_DATABASE=myappdb \
  -e MARIADB_USER=myapp \
  -e MARIADB_PASSWORD=mypassword \
  -v mariadb_data:/var/lib/mysql \
  -p 3306:3306 \
  mariadb:11

# Connect
docker exec -it mysql mysql -u myapp -p myappdb

# Backup
docker exec mysql mysqldump -u root -p myappdb > backup.sql

# Restore
docker exec -i mysql mysql -u root -p myappdb < backup.sql
```

### MongoDB

```bash
# Basic MongoDB
docker run -d \
  --name mongo \
  -p 27017:27017 \
  mongo:6

# With authentication
docker run -d \
  --name mongo \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=adminpassword \
  -e MONGO_INITDB_DATABASE=myappdb \
  -v mongo_data:/data/db \
  -v ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro \
  -p 27017:27017 \
  --restart unless-stopped \
  mongo:6

# Connect with mongosh
docker exec -it mongo mongosh -u admin -p adminpassword

# MongoDB with replica set (for transactions)
docker run -d \
  --name mongo \
  -v mongo_data:/data/db \
  -p 27017:27017 \
  mongo:6 --replSet rs0

# Initialize replica set
docker exec -it mongo mongosh --eval "rs.initiate()"
```

```javascript
// init-mongo.js
db = db.getSiblingDB('myappdb');

db.createUser({
  user: 'myapp',
  pwd: 'mypassword',
  roles: [{ role: 'readWrite', db: 'myappdb' }]
});

db.createCollection('items');
```

### Redis

```bash
# Basic Redis
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine

# With persistence
docker run -d \
  --name redis \
  -v redis_data:/data \
  -p 6379:6379 \
  redis:7-alpine redis-server --appendonly yes

# With password
docker run -d \
  --name redis \
  -v redis_data:/data \
  -p 6379:6379 \
  redis:7-alpine redis-server --appendonly yes --requirepass mypassword

# With custom configuration
docker run -d \
  --name redis \
  -v redis_data:/data \
  -v ./redis.conf:/usr/local/etc/redis/redis.conf:ro \
  -p 6379:6379 \
  redis:7-alpine redis-server /usr/local/etc/redis/redis.conf

# Connect with redis-cli
docker exec -it redis redis-cli
docker exec -it redis redis-cli -a mypassword  # With auth

# Persistence modes:
# RDB  - Point-in-time snapshots (default)
# AOF  - Append-only file (--appendonly yes)
# Both - Maximum durability
```

### Development Stack Example

```yaml
# docker-compose.yml - Complete development stack
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://myapp:password@postgres:5432/myappdb
      REDIS_URL: redis://redis:6379
      MONGO_URL: mongodb://mongo:27017/myappdb
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
      mongo:
        condition: service_started

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: myapp
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myappdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

  mongo:
    image: mongo:6
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

  # Database admin tools
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - mysql

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
    depends_on:
      - mongo

volumes:
  postgres_data:
  redis_data:
  mongo_data:
```

### Data Persistence

```bash
# CRITICAL: Always use volumes for database data

# Named volume (recommended)
docker run -d \
  --name postgres \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15

# List volumes
docker volume ls

# Inspect volume
docker volume inspect postgres_data

# Backup volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_backup.tar.gz /data

# Restore volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_backup.tar.gz -C /

# WITHOUT VOLUMES = DATA LOSS
# Container removed → Data gone!
docker rm postgres  # All data lost if no volume!
```

### Performance Considerations

```yaml
# Production database considerations:

performance_tips:
  - Use volumes (not bind mounts) for data
  - Set appropriate memory limits
  - Use dedicated networks
  - Consider tmpfs for temp tables
  - Monitor with tools like cAdvisor

# Memory limits
docker run -d \
  --name postgres \
  --memory=2g \
  --memory-swap=2g \
  postgres:15

# Shared memory (PostgreSQL)
docker run -d \
  --name postgres \
  --shm-size=256m \
  postgres:15

# Production reality:
# - Development/Testing: Docker is great
# - Production: Consider managed services (RDS, Cloud SQL)
# - Or: Run on dedicated VMs with proper ops
# - Stateful workloads in containers are complex
```

## Interview Questions

### Q1: Why is it important to use volumes with database containers?
**A:** Container filesystems are ephemeral - data is lost when the container is removed. Volumes persist data outside the container lifecycle. Without volumes, removing a database container destroys all data.

### Q2: What is the difference between named volumes and bind mounts for databases?
**A:** Named volumes are managed by Docker, stored in Docker's directory, and portable. Bind mounts map to specific host paths. Named volumes are preferred for databases as they're managed by Docker and work across platforms.

### Q3: How do you initialize a PostgreSQL database with schema on first run?
**A:** Mount SQL files to `/docker-entrypoint-initdb.d/`. Scripts in this directory run automatically on first container start when the data directory is empty. Example: `-v ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro`

### Q4: Should you run production databases in Docker containers?
**A:** For development and testing, Docker databases are excellent. For production, consider managed services (RDS, Cloud SQL) which handle backups, replication, and maintenance. If containerizing, use proper orchestration (Kubernetes with StatefulSets) and robust backup strategies.

### Q5: How do you backup a database running in Docker?
**A:** Use the database's backup tools via `docker exec`: `docker exec postgres pg_dump -U user dbname > backup.sql`. For volume-level backups, stop the container and backup the volume data.

### Q6: What is the purpose of `--shm-size` with PostgreSQL?
**A:** PostgreSQL uses shared memory for certain operations. Docker's default shared memory (64MB) may be insufficient. `--shm-size=256m` increases shared memory to prevent "could not resize shared memory segment" errors.

### Q7: How do you connect an application to a database in another container?
**A:** Use Docker networks. Containers on the same network can reach each other by container name. Connection string: `postgres://user:pass@container_name:5432/db`. With Compose, this networking is automatic.

### Q8: What is the `depends_on` with `condition` used for in Compose?
**A:** It ensures service startup order and health. `condition: service_healthy` waits for the database's healthcheck to pass before starting dependent services, preventing connection errors during startup.
