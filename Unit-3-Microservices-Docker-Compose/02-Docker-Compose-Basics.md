# Docker Compose Basics

## Table of Contents
1. [Introduction to Docker Compose](#introduction-to-docker-compose)
2. [YAML Structure](#yaml-structure)
3. [Writing docker-compose.yml](#writing-docker-composeyml)
4. [Version](#version)
5. [Services](#services)
6. [Volumes](#volumes)
7. [Networks](#networks)

---

## Introduction to Docker Compose

### What is Docker Compose?

**Definition**: Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file.

**Key Benefits:**
- Define entire application stack in a single file
- Start all services with one command
- Manage service dependencies
- Easy environment configuration
- Reproducible deployments

### Why Use Docker Compose?

**Without Docker Compose:**
```bash
# Start database
docker run -d --name db -e POSTGRES_PASSWORD=secret postgres

# Start backend
docker run -d --name backend --link db:database -p 3000:3000 backend-image

# Start frontend
docker run -d --name frontend --link backend:api -p 80:80 frontend-image
```

**With Docker Compose:**
```bash
docker-compose up
```

### Installation

**Linux:**
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

**Verify Installation:**
```bash
docker-compose --version
```

---

## YAML Structure

### YAML Basics

**YAML** (YAML Ain't Markup Language) is a human-readable data serialization format.

**Key Rules:**
- Indentation matters (use spaces, not tabs)
- Use 2 or 4 spaces for indentation
- Key-value pairs: `key: value`
- Lists use hyphens: `- item`
- Comments start with `#`

**Basic YAML Example:**
```yaml
# This is a comment
name: my-application
version: "3.8"

services:
  - web
  - database
  
configuration:
  port: 8080
  debug: true
```

### YAML Data Types

**Strings:**
```yaml
name: myapp
description: "This is a string with spaces"
multiline: |
  This is a
  multiline string
```

**Numbers:**
```yaml
port: 8080
replicas: 3
memory: 512
```

**Booleans:**
```yaml
debug: true
production: false
```

**Lists:**
```yaml
# Method 1
ports:
  - 8080
  - 9090

# Method 2
ports: [8080, 9090]
```

**Objects/Maps:**
```yaml
database:
  host: localhost
  port: 5432
  name: mydb
```

---

## Writing docker-compose.yml

### File Structure

**Basic Structure:**
```yaml
version: '3.8'

services:
  # Service definitions

volumes:
  # Volume definitions

networks:
  # Network definitions
```

### Complete Example

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

volumes:
  db-data:

networks:
  frontend:
  backend:
```

### File Location

**Standard Location:**
```
project/
├── docker-compose.yml
├── Dockerfile
├── app/
└── config/
```

**Custom File Name:**
```bash
docker-compose -f custom-compose.yml up
```

---

## Version

### Version Specification

**Purpose**: Defines the Compose file format version.

**Syntax:**
```yaml
version: '3.8'
```

### Version History

| Version | Docker Engine | Key Features |
|---------|---------------|--------------|
| 3.8 | 19.03.0+ | Latest features |
| 3.7 | 18.06.0+ | Init option |
| 3.6 | 18.02.0+ | tmpfs size |
| 3.5 | 17.12.0+ | Isolation mode |
| 3.4 | 17.09.0+ | Target stage |
| 3.3 | 17.06.0+ | Configs, secrets |
| 3.0 | 1.13.0+ | Major rewrite |
| 2.x | 1.10.0+ | Legacy format |

### Version Features

**Version 3.8 Features:**
```yaml
version: '3.8'

services:
  app:
    # CPU limits
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    
    # Init process
    init: true
    
    # Platform specification
    platform: linux/amd64
```

**Best Practice:**
- Use latest stable version (3.8)
- Check Docker Engine compatibility
- Version 3.x for production
- Version 2.x is legacy

---

## Services

### Service Definition

**Definition**: A service is a container configuration in Docker Compose.

**Basic Service:**
```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
```

### Service Configuration Options

#### 1. Image

**Using Pre-built Image:**
```yaml
services:
  web:
    image: nginx:alpine
  
  db:
    image: postgres:14
  
  redis:
    image: redis:7-alpine
```

#### 2. Build

**Building from Dockerfile:**
```yaml
services:
  app:
    build: .
  
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.dev
      args:
        - NODE_ENV=development
```

#### 3. Container Name

```yaml
services:
  web:
    container_name: my-web-server
    image: nginx
```

#### 4. Ports

**Port Mapping:**
```yaml
services:
  web:
    ports:
      - "8080:80"        # HOST:CONTAINER
      - "443:443"
      - "3000-3005:3000-3005"  # Range
```

**Expose (Internal Only):**
```yaml
services:
  api:
    expose:
      - "3000"  # Only accessible to other services
```

#### 5. Environment Variables

**Method 1: Inline**
```yaml
services:
  app:
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/mydb
      - DEBUG=true
```

**Method 2: Object**
```yaml
services:
  app:
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://db:5432/mydb
```

**Method 3: .env File**
```yaml
services:
  app:
    env_file:
      - .env
      - .env.local
```

#### 6. Depends On

**Service Dependencies:**
```yaml
services:
  web:
    depends_on:
      - api
      - cache
  
  api:
    depends_on:
      - db
  
  db:
    image: postgres
```

**With Conditions (v3.8+):**
```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
  
  db:
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
```

#### 7. Restart Policy

```yaml
services:
  app:
    restart: always
    # Options: no, always, on-failure, unless-stopped
```

#### 8. Command & Entrypoint

```yaml
services:
  app:
    command: npm start
  
  worker:
    command: ["python", "worker.py"]
  
  custom:
    entrypoint: /app/entrypoint.sh
```

#### 9. Working Directory

```yaml
services:
  app:
    working_dir: /app
```

#### 10. User

```yaml
services:
  app:
    user: "1000:1000"
```

### Complete Service Example

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend-api
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://db:5432/mydb
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - app-network
    depends_on:
      - db
    restart: unless-stopped
    command: npm start
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

## Volumes

### What are Volumes?

**Definition**: Persistent data storage that exists outside container lifecycle.

**Types:**
1. Named volumes (managed by Docker)
2. Bind mounts (host filesystem)
3. Anonymous volumes

### Named Volumes

**Definition:**
```yaml
version: '3.8'

services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:  # Named volume
```

**With Driver Options:**
```yaml
volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /path/on/host
      o: bind
```

### Bind Mounts

**Syntax:**
```yaml
services:
  web:
    volumes:
      - ./html:/usr/share/nginx/html  # HOST:CONTAINER
      - ./config:/etc/nginx/conf.d:ro  # Read-only
```

**Use Cases:**
- Development (live code reload)
- Configuration files
- Log files

### Anonymous Volumes

```yaml
services:
  app:
    volumes:
      - /app/node_modules  # Anonymous volume
```

### Volume Examples

**Database Persistence:**
```yaml
services:
  postgres:
    image: postgres:14
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  postgres-data:
```

**Development Setup:**
```yaml
services:
  app:
    volumes:
      - ./src:/app/src          # Source code
      - ./package.json:/app/package.json
      - /app/node_modules       # Don't override node_modules
```

**Multiple Volumes:**
```yaml
services:
  app:
    volumes:
      - app-data:/data
      - app-logs:/var/log
      - ./config:/etc/app:ro

volumes:
  app-data:
  app-logs:
```

### Volume Commands

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect myproject_db-data

# Remove volume
docker volume rm myproject_db-data

# Remove all unused volumes
docker volume prune
```

---

## Networks

### What are Networks?

**Definition**: Virtual networks that allow containers to communicate.

**Default Behavior:**
- Docker Compose creates a default network
- All services can communicate using service names

### Network Types

**1. Bridge (Default)**
```yaml
networks:
  app-network:
    driver: bridge
```

**2. Host**
```yaml
networks:
  host-network:
    driver: host
```

**3. Overlay (Swarm)**
```yaml
networks:
  overlay-network:
    driver: overlay
```

**4. None**
```yaml
services:
  isolated:
    network_mode: none
```

### Network Configuration

**Basic Network:**
```yaml
version: '3.8'

services:
  web:
    networks:
      - frontend
  
  api:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

**Network with Options:**
```yaml
networks:
  app-network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: my-bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
```

### Network Isolation Example

**Three-Tier Architecture:**
```yaml
version: '3.8'

services:
  # Frontend - only on frontend network
  nginx:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"
  
  # Backend - on both networks
  api:
    image: node:16
    networks:
      - frontend
      - backend
  
  # Database - only on backend network
  postgres:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

### Service Discovery

**Using Service Names:**
```yaml
services:
  web:
    environment:
      - API_URL=http://api:3000
      - DB_HOST=database
  
  api:
    # Can connect to 'database' by name
  
  database:
    # Accessible as 'database' to other services
```

### Network Aliases

```yaml
services:
  db:
    networks:
      backend:
        aliases:
          - database
          - postgres-db
```

### External Networks

**Using Existing Network:**
```yaml
networks:
  existing-network:
    external: true
    name: my-pre-existing-network
```

### Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect myproject_default

# Create network
docker network create my-network

# Remove network
docker network rm my-network

# Connect container to network
docker network connect my-network my-container
```

---

## Summary

**Docker Compose** provides:
- **YAML Structure**: Human-readable configuration format
- **Version**: Defines Compose file format and features
- **Services**: Container definitions with full configuration
- **Volumes**: Persistent data storage
- **Networks**: Container communication and isolation

**Key Commands:**
```bash
docker-compose up          # Start services
docker-compose down        # Stop and remove services
docker-compose ps          # List services
docker-compose logs        # View logs
docker-compose exec        # Execute command in service
```

This foundation enables building complex multi-container applications with simple, declarative configuration.
