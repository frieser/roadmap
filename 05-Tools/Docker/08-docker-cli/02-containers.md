---
tags: ['docker', 'containers', 'cli', 'devops', 'tools', 'roadmap']
---

# Docker Containers CLI

## Summary

Docker's container management commands provide full lifecycle control over running containers. From creating and starting containers to stopping, removing, and inspecting them, understanding these commands is essential for effective containerized application management. These commands operate on running containers, as opposed to images which are read-only templates.

## Detailed Explanation

### Listing Containers

```bash
# LIST RUNNING CONTAINERS
docker ps
# CONTAINER ID   IMAGE     COMMAND              CREATED       STATUS      PORTS     NAMES
# abc123         nginx:latest  "nginx -g 'daemon off;"  2 hours ago   Up         80/tcp    web

# LIST ALL CONTAINERS (INCLUDING STOPPED)
docker ps -a
# CONTAINER ID   IMAGE     COMMAND              CREATED       STATUS      PORTS     NAMES
# abc123         nginx:latest  "nginx -g 'daemon off;"  2 hours ago   Up         80/tcp    web
# def456         postgres:15  "postgres -D"           1 day ago    Exited (0) 5432/tcp   db

# LIST ONLY CONTAINER IDS
docker ps -q
# abc123
# def456

# LIST ONLY NAMES
docker ps --format '{{.Names}}'
# web
# db

# CUSTOM FORMATTING
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
docker ps --format "{{.ID}}: {{.Names}}"
docker ps --format json | jq '.[] | {id: .Id, name: .Names[0], image: .Image}'
```

### Creating and Starting Containers

```bash
# RUN (CREATE AND START)
docker run nginx
# Creates new container from nginx:latest and starts it
# Same as: docker create nginx && docker start $(docker ps -lq -q)

# RUN WITH NAME
docker run --name myweb nginx
docker run --name mydb postgres:15

# RUN IN BACKGROUND (DETACHED)
docker run -d nginx
docker run --detach nginx

# RUN INTERACTIVELY
docker run -it ubuntu:22.04 bash
docker run --interactive --tty ubuntu:22.04 bash

# RUN WITH CUSTOM COMMAND
docker run nginx echo "Hello World"
docker run ubuntu:22.04 cat /etc/os-release

# RUN WITH PORTS
docker run -p 8080:80 nginx
docker run --publish 8080:80 nginx

# RUN WITH VOLUME
docker run -v mydata:/app/data nginx

# RUN WITH ENVIRONMENT
docker run -e APP_ENV=production nginx

# RUN AND REMOVE ON EXIT
docker run --rm nginx

# AUTO REMOVE ON EXIT (with custom name)
docker run --rm --name temp nginx echo "test"
# Container removed after exit, but name already used
```

### Stopping and Restarting Containers

```bash
# STOP CONTAINER
docker stop myweb
# Sends SIGTERM, allows graceful shutdown
# Container status changes from Up to Exited

# STOP MULTIPLE CONTAINERS
docker stop myweb mydb mycache

# FORCE STOP (SIGKILL)
docker kill myweb
# Sends SIGKILL, immediate termination
docker kill -s SIGINT myweb  # Send specific signal

# RESTART CONTAINER
docker restart myweb
# docker stop && docker start

# RESTART MULTIPLE
docker restart myweb mydb

# RESTART WITH TIMEOUT
docker restart -t 30 myweb  # Wait 30s before declaring failure
```

### Removing Containers

```bash
# REMOVE CONTAINER (MUST BE STOPPED)
docker rm myweb

# FORCE REMOVE (RUNNING)
docker rm -f myweb
# docker stop && docker rm

# REMOVE MULTIPLE
docker rm myweb mydb mycache

# REMOVE ALL STOPPED CONTAINERS
docker container prune  # Remove all stopped containers

# REMOVE BY TIME
docker container prune --filter "until=24h"  # Containers stopped more than 24h ago

# AUTO REMOVE ON EXIT (DIDN'T WORK ALREADY)
docker run --rm nginx  # Already handles removal

# REMOVE AND ASSOCIATED ANONYMOUS VOLUMES
docker rm -v myweb
# Removes container and anonymous volumes
```

### Inspecting Containers

```bash
# DETAILED INSPECTION
docker inspect myweb
# Shows full JSON configuration

# INSPECT SPECIFIC FIELDS
docker inspect --format='{{.State.Status}}' myweb
# Up

docker inspect --format='{{.State.StartedAt}}' myweb
# 2024-01-09T12:00:00Z

docker inspect --format='{{range .Config.Env}}{{.Key}}={{.Value}}{{"\n"}}{{end}}' myweb
# DATABASE_URL=postgres://localhost/db
# LOG_LEVEL=info
# NODE_ENV=production

# INSPECT MULTIPLE CONTAINERS
docker inspect myweb mydb

# INSPECT RUNNING PROCESSES
docker top myweb
# UID                 PID    PPID    C    STIME   TTY       TIME       COMMAND
# root                1     0       0       00:00:00   nginx: master
```

### Container Logs

```bash
# SHOW ALL LOGS
docker logs myweb
# Shows all log output

# FOLLOW LOGS (STREAMING)
docker logs -f myweb
# Continuously show new logs, use Ctrl+C to exit

# FOLLOW WITH TAIL (LAST N LINES)
docker logs --tail 100 myweb
# Show last 100 lines

# FOLLOW WITH TIMESTAMP
docker logs -t myweb
# Shows timestamps on each line

# SHOW LAST N LINES
docker logs --tail 50 myweb

# SHOW SINCE SPECIFIC TIME
docker logs --since 2024-01-09T10:00:00 myweb

# SHOW UNTIL SPECIFIC TIME
docker logs --until 2024-01-09T12:00:00 myweb

# SHOW LOGS WITH DETAILS
docker logs --details myweb

# MULTIPLE CONTAINER LOGS
docker logs myweb mydb
docker logs --tail 10 --since 5m myweb

# LOGS WITH COLOR
docker logs --color=always myweb

# QUIET MODE (NO HEADERS)
docker logs --quiet myweb
```

### Executing Commands in Containers

```bash
# RUN COMMAND INTERACTIVELY
docker exec -it myweb bash
docker exec --interactive --tty myweb sh

# RUN SINGLE COMMAND
docker exec myweb cat /etc/nginx/nginx.conf
docker exec myweb ls -la /app

# RUN AS DIFFERENT USER
docker exec -u nginx myweb whoami

# RUN COMMAND IN SPECIFIC DIRECTORY
docker exec -w /app myweb ls
docker exec --workdir /tmp myweb pwd

# EXECUTE SCRIPT
docker exec myweb sh -c 'echo "Hello" > /tmp/test.txt'

# EXECUTE AS ROOT
docker exec -u root myweb bash

# DETACHED EXECUTION (NON-INTERACTIVE)
docker exec -d myweb python script.py

# ACCESSING CONTAINER SHELL FROM HOST
docker exec -it myweb /bin/bash
```

### Container Stats and Resources

```bash
# LIVE STATISTICS STREAMING
docker stats
# Shows CPU, memory, network, disk I/O for all running containers
# Updates every second

# STATS FOR SPECIFIC CONTAINER
docker stats myweb
# CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O   PIDS
# abc123         myweb     0.50%    512MiB / 1GiB        50.25%      0B / 0B    15

# STATS WITH NO STREAM
docker stats --no-stream myweb

# FORMATTED STATS
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# CUSTOM FORMATTING
docker stats --format json | jq '.[] | {name: .Name, cpu: .CPUPerc, memory: .MemUsage}'
```

### Container Processes

```bash
# LIST PROCESSES IN CONTAINER
docker ps myweb
# Shows process in header (single process)

# LIST ALL PROCESSES
docker top myweb
# Shows all processes (including forks)

# LIST PROCESSES WITH DETAILS
docker top myweb -eo pid,ppid,user,stat,cmd

# CONTINUOUSLY MONITOR PROCESSES
docker top myweb  # Updates every second, Ctrl+C to exit

# LIST PROCESSES WITH SPECIFIC FORMAT
docker top --format "table {{.PID}}\t{{.USER}}\t{{.COMMAND}}" myweb
```

### Container Diff and Changes

```bash
# SHOW FILES ADDED, CHANGED, DELETED
docker diff myweb
# C /app/newfile.txt       (Added)
# D /app/config.json        (Changed)
# D /tmp/oldfile.txt       (Deleted)

# DIFF WITH STATS
docker diff --stat myweb
# Shows statistics about changes

# EXPORT CONTAINER FILESYSTEM
docker export myweb > myweb-backup.tar
# Exports container filesystem to tar file
tar -tf myweb-backup.tar  # View contents

# COPY FILES FROM CONTAINER
docker cp myweb:/app/config.json ./config.json
docker cp myweb:/app/ ./app-backup  # Copy directory

# COPY FILES TO CONTAINER
docker cp ./config.json myweb:/app/config.json
docker cp ./app-backup/ myweb:/app/

# COPY BETWEEN CONTAINERS
docker cp myweb:/app/data.txt - | docker exec -i mydb sh -c 'cat > /data/data.txt'
```

### Container Update Commands

```bash
# UPDATE CONFIGURATION (LIMITED FUNCTIONALITY)
docker update --restart=always myweb
docker update --memory=1g --memory-swap=2g myweb
docker update --cpus=2.0 myweb

# UPDATE ENVIRONMENT VARIABLES
docker update -e NEW_VAR=value myweb
docker update --env-file ./new-env-vars.env myweb

# UPDATE ALL CONTAINERS WITH SAME IMAGE
docker update --restart=always $(docker ps -q --filter ancestor=myapp:latest)
```

### Container Wait Commands

```bash
# WAIT FOR CONTAINER TO EXIT
docker wait myweb
# Returns exit code when container stops

# WAIT FOR MULTIPLE CONTAINERS
docker wait myweb mydb
docker wait $(docker ps -q)  # Wait for all to exit

# WAIT WITH TIMEOUT (Bash)
timeout 30s docker wait myweb  # Wait max 30s

# PRACTICAL EXAMPLE
# Start database
docker run -d --name db postgres:15
# Wait for database to be ready
docker run -d --name app myapp:latest --link db:db
# Wait for app to complete
docker wait app
# Clean up
docker stop db app && docker rm -f db app
```

## Interview Questions

### Q1: What is the difference between `docker stop` and `docker kill`?
**A:** `docker stop` sends SIGTERM signal for graceful shutdown, allowing the container to clean up and exit properly. `docker kill` sends SIGKILL for immediate termination without cleanup. Use `stop` for normal shutdown, `kill` only when container is unresponsive.

### Q2: How do you remove a container that's still running?
**A:** Use `docker rm -f <container>` to force remove a running container. This sends SIGKILL, immediately terminating the container. Without `-f`, you must stop the container first with `docker stop`.

### Q3: What is the purpose of `docker exec` vs `docker run`?
**A:** `docker run` creates a new container from an image and starts it. `docker exec` runs a command inside an existing, running container. Use `run` for new containers, `exec` for debugging or one-off commands in running containers.

### Q4: How do you follow container logs in real-time?
**A:** Use `docker logs -f <container>` which follows logs and streams new output to your terminal. Add `--tail` to show only recent lines (e.g., `--tail 50`) or `--since` to limit by time.

### Q5: What does `docker stats` show?
**A:** `docker stats` displays real-time resource usage (CPU, memory, network, disk I/O, PIDs) for all running containers. Add `--no-stream` to get a single snapshot, or specific containers as arguments.

### Q6: What is the difference between `docker cp` and `docker export`?
**A:** `docker cp` copies specific files or directories between container and host. `docker export` exports the entire container filesystem to a tar archive. Use `cp` for selective file transfers, `export` for full backups.

### Q7: What does `docker wait` do?
**A:** `docker wait <container>` blocks until the container exits and returns the container's exit code. This is useful for waiting for initialization processes or coordinating container startup sequences.

### Q8: How do you update a running container's configuration?
**A:** Use `docker update` to modify container settings without recreating it. You can change restart policies, resource limits, environment variables, and more. For example: `docker update --restart=always myweb`.
