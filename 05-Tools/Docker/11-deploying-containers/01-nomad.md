---
tags: ['docker', 'containers', 'deployment', 'devops', 'kubernetes', 'nomad', 'tools', 'roadmap']
---

# Nomad

## Summary

Nomad is an orchestration and scheduler platform by HashiCorp that uses a declarative job specification format. It provides features for service discovery, constraint-based placement, health checks, rolling deployments, and multi-datacenter awareness. Nomad runs on Linux, macOS, and Windows, making it a flexible alternative to Kubernetes for smaller deployments or workloads that don't require Kubernetes complexity.

## Detailed Explanation

### Nomad Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      NOMAD SERVER                              │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Client                    │  Consul    │  │
│  │  HTTP API                 │  KV Store   │  │
│  │  Server                │  Server    │  │
│  │    │  └─────────────────┘  │
│  └──────────────────────────────────────────────────┘  │
│                         │                          │       │
│                         │                          │       │
│                         ▼                          │       │
│              ┌───────────────────────┐         │       │
│              │  SCHEDULER        │         │       │
│              │  Evaluates jobs on   │         │       │
│              │  constraints:        │         │       │
│              └───────────────────────┘         │       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                   ┌────────────────────────────────────────────┐
                   │  CLIENT AGENTS                        │
                   │  • Nomad Client (CLI/GUI)              │
                   │  • Docker / Podman                      │
                   │  • Consul / Vault                      │
                   │  • Nomad Autoscaler                     │
                   └─────────────────────────────────────────────┘
                              │
                              ▼
                 ┌──────────────────────────────────────────────┐
                 │  EXECUTED JOBS                          │
                 │  • Drivers (QEMU, Docker, ...)          │
                 │  • Tasks (Shell scripts, Docker)         │
                 │  └─────────────────────────────────────────────┘
```

### Nomad Job Specification

```hcl
# Basic Nomad Job
job "web-app" {
  datacenters = ["dc1"]
  type = "service"
  
  task "app" {
    driver = "docker"
    config {
      image = "myapp:latest"
      port_map = "http:8080:80"
      
      resources {
        cpu    = 500
        memory = 512
      }
    }
  }
}

# Job with multiple tasks
job "full-stack" {
  datacenters = ["dc1"]
  type = "service"
  
  group "app" {
    count = 3
    
    task "web" {
      driver = "docker"
      config {
        image = "myapp:latest"
      port_map = "http:8080:80"
      }
    }
    
    task "nginx" {
      driver = "docker"
      config {
        image = "nginx:alpine"
        port_map = "80:80"
        resources {
          cpu = 100
          memory = 256
        }
      }
    }
  }
  
  group "worker" {
    count = 5
    
    task "worker" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }
    }
  }
}

# Job with constraints
job "database" {
  type = "service"
  datacenters = ["dc1", "dc2"]  # Only run in specific datacenters
  
  constraint {
    attribute = "${attr.kernel}"
    value = "true"
  }
  
  task "postgres" {
    driver = "docker"
    config {
      image = "postgres:15"
      port_map = "5432:5432"
    }
  }
}
```

### Service Discovery

```hcl
# NOMAD SERVICE WITH CONSUL
# Consul automatically registers services
service "web-app" {
  name = "myapp"
  tags = ["api", "http"]
  port = 8080
  
  check {
    type = "tcp"
    interval = "10s"
    timeout = "2s"
  }
}

# SERVICE WITH HEALTH CHECK
service "api" {
  name = "myapp"
  
  check {
    type = "http"
    path = "/health"
    interval = "15s"
    timeout = "3s"
    failures_before_critical = 3
  }
  
  connect {
    sidecar_service = "nginx"
  }
}
```

### Volume and Storage

```hcl
# NOMAD SERVICE WITH HOST VOLUME
service "database" {
  
  task "postgres" {
    driver = "docker"
    
    config {
      image = "postgres:15"
      
      volume_mount {
        volume      = "database-data"
        destination = "/var/lib/postgresql/data"
        read_only  = false
      }
    }
  }
  
  # Volume with mount options
  service "app" {
    task "app" {
      driver = "docker"
      
      config {
        image = "myapp:latest"
        
        volume_mount {
          volume      = "app-logs"
          destination = "/var/log/app"
          read_only = true
        }
      }
    }
  }
}

# CSI PLUGIN VOLUME
# Example with CSI volume driver
service "app" {
  task "app" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      volume_mount {
        type      = "csi"
        driver     = "csi-hostpath"
        volume_id   = "myapp-data"
        destination = "/data"
      }
    }
  }
}
```

### Networking

```hcl
# HOST NETWORKING
service "web-app" {
  
  task "app" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      network_mode = "host"
      
      port_map {
        http = 8080
      }
    }
  }
}

# BRIDGE NETWORKING
service "api" {
  
  task "api" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      network_mode = "bridge"
      
      port_map {
        api = 8000
      }
    }
  }
}

# MULTIPLE NETWORK INTERFACES
service "web-app" {
  
  task "app" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      network_mode = "bridge"
      
      port_map {
        http  = 8080
        https = 8443
      }
    }
  }
}

# CNI PLUGIN (CUSTOM NETWORK)
# Requires Nomad with CNI plugin
service "app" {
  
  task "app" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      network {
        mode = "cni/${attr.cni}"
        ip = "10.0.10.100"
      }
    }
  }
}
```

### Environment Variables

```hcl
# ENVIRONMENT VARIABLES
job "web-app" {
  
  task "app" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      
      env {
        DATABASE_URL = "postgres://db:5432/app"
        REDIS_HOST   = "redis:6379"
        
        # Vault secrets
        DATABASE_PASSWORD = "{{ with \"nomad\" \"database\" \"password\" }}"
      }
    }
  }
}

# FROM VAULT
job "app" {
  task "app" {
    driver = "docker"
    
    vault {
      policies       = "default"
      change_mode   = "no-snapshot"
      
      template {
        source      = "secret/database/password"
        destination = "DATABASE_PASSWORD"
      }
    }
  }
}

# NOMAD CLI - SET VARIABLES
nomad job run \
  -var database_url=postgres://db:5432/app \
  -var redis_host=redis:6379 \
  web-app
```

### Constraints

```hcl
# CONSTRAINT EXAMPLES
# Only run on specific node
job "web-app" {
  datacenters = ["dc1", "dc2"]
  
  constraint {
    attribute = "${node.class}"
    value     = "web"
    operator = "!="
  }
}

# Must run with attribute
job "web-app" {
  datacenters = ["dc1"]
  
  constraint {
    attribute = "${node.class}"
    value     = "compute"
    operator = "="
  }
}

# VERSION CONSTRAINTS
job "web-app" {
  
  constraint {
    attribute = "${attr.version}"
    value     = "1.2.0"
  }
  
  constraint {
    attribute = "${attr.kernel}"
    value     = "true"
  }
}

# OPERATORS
# AND: Both conditions must be true
constraint {
  attribute = "${attr.kernel}"
    value     = "true"
  operator = "AND"
}

# OR: At least one condition must be true
constraint {
  attribute = "${attr.kernel}"
    value     = "true"
  operator = "OR"
}
```

### Rolling Updates

```hcl
# AUTOMATIC ROLLING
job "web-app" {
  
  update {
    max_parallel      = 2    # Two at a time
    health_check      = "checks"
    min_healthy_time   = "10s"
    healthy_deadline   = "5m"
    canary            = 1    # 1 canary before all
  }
}

# BLUE-GREEN DEPLOYMENT
group "production" {
  count = 10
  
  task "web-app" {
    driver = "docker"
    
    version = "v1.0.0-green"    # Canary version
    count = 2
    
    version = "v1.0.0"            # Stable version
    count = 10
  }
  
  update {
    max_parallel = 2
    health_check = "checks"
    auto_promote = true
    auto_revert = false
  }
}
```

### Scaling

```hcl
# DYNAMIC SCALING WITH AUTOSCALER
autoscaler {
  job "web-app" {
    min = 1
    max = 10
    
    policy {
      evaluation_interval = "30s"
      cooldown           = "5m"
    }
  }
}

# CRON-BASED SCALING
job "scale-up" {
  type = "batch"
  schedule = "0 9 * * *"  # Every morning at 9am
    
  task "web-app" {
    driver = "docker"
    
    count = 5    # Scale up to 5 instances
    
    config {
      image = "myapp:latest"
    }
  }
}

job "scale-down" {
  type = "batch"
  schedule = "0 20 * * *"  # Every evening at 8pm
  
  task "web-app" {
    driver = "docker"
    
    count = 3    # Scale down to 3 instances
  }
  }
}
```

### System Jobs (Batch Jobs)

```hcl
# CRON JOB - BACKUP DATABASE
job "backup-db" {
  type = "batch"
  schedule = "0 2 * * *"  # Daily at 2am
  
  task "backup" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      command = "python backup.py"
    }
  }
}

# ONE-TIME JOB
job "migration" {
  type = "batch"
  
  task "migrate" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      command = "python migrate.py"
    }
  }
}

# PERIODIC JOB
job "cleanup" {
  type = "batch"
  schedule = "@weekly"
  
  task "cleanup" {
    driver = "docker"
    
    config {
      image = "myapp:latest"
      command = "python cleanup_old_logs.py"
    }
  }
}
```

## Interview Questions

### Q1: What is Nomad and how does it differ from Kubernetes?
**A:** Nomad is HashiCorp's orchestration platform that uses declarative job specifications (HCL). It's simpler than Kubernetes, easier to operate, and better for smaller deployments. Kubernetes has a more complex architecture with multiple controllers (kubelet, kube-proxy, etc.) and is better for very large-scale deployments. Nomad is often described as "Kubernetes without the complexity."

### Q2: What are the main components of Nomad?
**A:** Nomad consists of Nomad server, Client agents (CLI/GUI), data storage (Consul, Vault), and drivers. The scheduler evaluates jobs based on constraints and resources, dispatches tasks to clients, and manages their lifecycle. Drivers run workloads (Docker containers, QEMU VMs, etc.).

### Q3: How do you deploy applications with Nomad?
**A:** Write job specifications in HCL format defining your application (image, resources, constraints). Use `nomad job run` to submit jobs to the cluster. Nomad will schedule the jobs, handle placement, and manage the containers throughout their lifecycle.

### Q4: What is the difference between a task and a group?
**A:** A task is the smallest schedulable unit - a single work instance. A group is a collection of tasks that scale together. Groups can be controlled by `count` and updated as a unit, enabling rolling deployments where you define count per group (e.g., 10 canary + 90 stable).

### Q5: How does service discovery work in Nomad?
**A:** Nomad integrates with Consul for service discovery. Services register their health check endpoints and are made discoverable via DNS or API. Clients query Consul or use Nomad's service discovery to find services. Alternatively, you can use static port configuration with `service` blocks.

### Q6: What are the main types of jobs in Nomad?
**A:** `service` jobs run continuously until stopped, `batch` jobs run once and exit, `sysbatch` jobs run on a schedule (cron-like). `system` jobs run on the local node (rarely used). Use `service` for long-running applications and `batch` for periodic tasks.

### Q7: How do you implement rolling deployments with Nomad?
**A:** Use canary deployments with multiple job versions. Deploy a canary version (count=1) alongside stable (count=9). Monitor the canary, and if healthy, auto-promote to full deployment. Use `update { max_parallel, auto_promote }` to control rollout speed and canary fail behavior.

### Q8: How do you use environment variables and secrets with Nomad?
**A:** Define `env` blocks in jobs for environment variables. Use Vault integration for secrets management with `vault { policies, change_mode, template }` to inject secrets at runtime. Never hardcode secrets in job specifications.
