# Advanced Docker Compose

## Table of Contents
1. [Environment Variables](#environment-variables)
2. [Secrets and Configs](#secrets-and-configs)
3. [Build vs Image Fields](#build-vs-image-fields)
4. [Service Dependency Ordering](#service-dependency-ordering)

---

## Environment Variables

### What are Environment Variables?

**Definition**: Key-value pairs that configure application behavior without changing code.

**Use Cases:**
- Database credentials
- API keys
- Feature flags
- Environment-specific settings (dev/staging/prod)

### Methods to Set Environment Variables

#### 1. Inline in docker-compose.yml

**Array Format:**
```yaml
services:
  app:
    image: myapp
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://db:5432/mydb
      - API_KEY=secret123
      - DEBUG=false
```

**Object Format:**
```yaml
services:
  app:
    image: myapp
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://db:5432/mydb
      API_KEY: secret123
      DEBUG: false
```

#### 2. Using .env File

**Create .env file:**
```bash
# .env
NODE_ENV=production
DATABASE_URL=postgresql://db:5432/mydb
API_KEY=secret123
PORT=3000
```

**Reference in docker-compose.yml:**
```yaml
services:
  app:
    image: myapp
    env_file:
      - .env
```

**Multiple .env Files:**
```yaml
services:
  app:
    env_file:
      - .env          # Common variables
      - .env.local    # Local overrides
      - .env.prod     # Production specific
```

#### 3. Variable Substitution

**Using Host Environment Variables:**
```yaml
services:
  app:
    image: myapp:${TAG:-latest}
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - API_KEY=${API_KEY}
    ports:
      - "${PORT:-3000}:3000"
```

**Host .env file:**
```bash
# .env (on host)
TAG=v1.2.3
DATABASE_URL=postgresql://prod-db:5432/mydb
API_KEY=prod-key-123
PORT=8080
```

**Run with substitution:**
```bash
docker-compose up
```

#### 4. Shell Environment Variables

```bash
# Set in shell
export DATABASE_URL=postgresql://localhost:5432/mydb
export API_KEY=my-secret-key

# Use in compose
docker-compose up
```

### Environment Variable Precedence

**Priority (highest to lowest):**
1. Compose file `environment` section
2. Shell environment variables
3. `.env` file
4. Dockerfile `ENV`
5. Default values

**Example:**
```yaml
# docker-compose.yml
services:
  app:
    environment:
      - PORT=3000  # Highest priority
    env_file:
      - .env       # Lower priority
```

### Advanced Environment Variable Patterns

#### Default Values

```yaml
services:
  app:
    environment:
      - PORT=${PORT:-3000}           # Default to 3000
      - NODE_ENV=${NODE_ENV:-development}
      - LOG_LEVEL=${LOG_LEVEL:-info}
```

#### Required Variables

```yaml
services:
  app:
    environment:
      - DATABASE_URL=${DATABASE_URL?Database URL is required}
      - API_KEY=${API_KEY?API Key must be set}
```

#### Computed Variables

```yaml
services:
  app:
    environment:
      - APP_URL=http://${HOST:-localhost}:${PORT:-3000}
      - DATABASE_URL=postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
```

### Complete Example

**.env file:**
```bash
# Application
NODE_ENV=production
APP_NAME=MyApp
APP_VERSION=1.0.0

# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=mydb
DB_USER=admin
DB_PASSWORD=secret123

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# API Keys
JWT_SECRET=my-jwt-secret
API_KEY=external-api-key
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    env_file:
      - .env
    environment:
      # Override or add specific variables
      - PORT=3000
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
      - REDIS_URL=redis://${REDIS_HOST}:${REDIS_PORT}
    ports:
      - "${PORT:-3000}:3000"
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}

  redis:
    image: redis:7-alpine
```

### Best Practices

✅ **Never commit secrets to version control**
```bash
# .gitignore
.env
.env.local
.env.*.local
```

✅ **Use .env.example for documentation**
```bash
# .env.example
NODE_ENV=development
DATABASE_URL=postgresql://localhost:5432/mydb
API_KEY=your-api-key-here
```

✅ **Validate required variables**
```yaml
environment:
  - DATABASE_URL=${DATABASE_URL?Required}
```

✅ **Use meaningful names**
```bash
# Good
DATABASE_URL=postgresql://...
JWT_SECRET=...

# Bad
DB=postgresql://...
SECRET=...
```

---

## Secrets and Configs

### Docker Secrets

**Definition**: Secure way to store sensitive data (passwords, certificates, API keys) in Docker Swarm.

**Note**: Secrets are primarily for Docker Swarm mode, but can be simulated in Compose.

#### Using Secrets in Docker Swarm

**Create secret:**
```bash
echo "my-secret-password" | docker secret create db_password -
```

**docker-compose.yml (Swarm mode):**
```yaml
version: '3.8'

services:
  db:
    image: postgres:14
    secrets:
      - db_password
      - db_user
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER_FILE: /run/secrets/db_user

secrets:
  db_password:
    external: true
  db_user:
    external: true
```

#### Secrets from Files (Compose)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    image: myapp
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

**Secrets are mounted at:**
```
/run/secrets/db_password
/run/secrets/api_key
```

**Application code:**
```javascript
// Node.js example
const fs = require('fs');
const dbPassword = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();
```

### Docker Configs

**Definition**: Non-sensitive configuration data that can be shared across services.

**docker-compose.yml (Swarm mode):**
```yaml
version: '3.8'

services:
  web:
    image: nginx
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
      - source: site_config
        target: /etc/nginx/conf.d/default.conf

configs:
  nginx_config:
    file: ./nginx.conf
  site_config:
    file: ./site.conf
```

### Secrets vs Environment Variables vs Configs

| Feature | Environment Variables | Secrets | Configs |
|---------|----------------------|---------|---------|
| **Use Case** | Non-sensitive config | Passwords, keys | Configuration files |
| **Storage** | Plain text | Encrypted | Plain text |
| **Access** | All processes | Mounted files | Mounted files |
| **Rotation** | Restart required | Can rotate | Can update |
| **Best For** | Public settings | Credentials | Config files |

### Complete Secrets Example

**Project Structure:**
```
project/
├── docker-compose.yml
├── secrets/
│   ├── db_password.txt
│   ├── jwt_secret.txt
│   └── api_key.txt
└── configs/
    └── app_config.json
```

**secrets/db_password.txt:**
```
my-secure-database-password
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    secrets:
      - db_password
      - jwt_secret
      - api_key
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
      - API_KEY_FILE=/run/secrets/api_key
    depends_on:
      - db

  db:
    image: postgres:14
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: mydb

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  api_key:
    file: ./secrets/api_key.txt
```

**Application Code (Node.js):**
```javascript
const fs = require('fs');

function readSecret(secretName) {
  try {
    return fs.readFileSync(`/run/secrets/${secretName}`, 'utf8').trim();
  } catch (error) {
    console.error(`Failed to read secret: ${secretName}`);
    process.exit(1);
  }
}

const dbPassword = readSecret('db_password');
const jwtSecret = readSecret('jwt_secret');
const apiKey = readSecret('api_key');

// Use secrets in application
const dbConfig = {
  host: 'db',
  password: dbPassword,
  database: 'mydb'
};
```

### Best Practices for Secrets

✅ **Never commit secrets to Git**
```bash
# .gitignore
secrets/
*.key
*.pem
.env
```

✅ **Use secret management tools in production**
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager

✅ **Rotate secrets regularly**

✅ **Use least privilege access**

✅ **Audit secret access**

---

## Build vs Image Fields

### Understanding Build and Image

**Two ways to define service containers:**
1. **image**: Use pre-built image from registry
2. **build**: Build image from Dockerfile

### Using Image Field

**Syntax:**
```yaml
services:
  web:
    image: nginx:alpine
```

**Use Cases:**
- Official images (nginx, postgres, redis)
- Third-party images
- Pre-built custom images
- Production deployments

**Examples:**
```yaml
services:
  # Official image
  nginx:
    image: nginx:1.21-alpine
  
  # Database
  postgres:
    image: postgres:14
  
  # Custom image from registry
  app:
    image: myregistry.com/myapp:v1.2.3
  
  # Docker Hub image
  redis:
    image: redis:7-alpine
```

### Using Build Field

**Simple Build:**
```yaml
services:
  app:
    build: .  # Build from Dockerfile in current directory
```

**Build with Context:**
```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
```

**Advanced Build Configuration:**
```yaml
services:
  app:
    build:
      context: ./app              # Build context directory
      dockerfile: Dockerfile.dev  # Custom Dockerfile name
      args:                       # Build arguments
        - NODE_ENV=development
        - APP_VERSION=1.0.0
      target: development         # Multi-stage build target
      cache_from:                 # Cache sources
        - myapp:latest
      labels:                     # Image labels
        - "com.example.version=1.0"
      shm_size: '2gb'            # Shared memory size
```

### Build and Image Together

**Build and Tag:**
```yaml
services:
  app:
    build: ./app
    image: myapp:latest  # Tag the built image
```

**Benefits:**
- Built image is tagged
- Can push to registry
- Reusable across compose files

### Build Arguments

**Dockerfile:**
```dockerfile
FROM node:16-alpine

ARG NODE_ENV=production
ARG APP_VERSION=1.0.0

ENV NODE_ENV=${NODE_ENV}
ENV APP_VERSION=${APP_VERSION}

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

CMD ["npm", "start"]
```

**docker-compose.yml:**
```yaml
services:
  app:
    build:
      context: .
      args:
        NODE_ENV: development
        APP_VERSION: ${APP_VERSION:-1.0.0}
```

### Multi-Stage Builds

**Dockerfile:**
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/index.js"]

# Development stage
FROM node:16 AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

**docker-compose.yml:**
```yaml
services:
  # Development
  app-dev:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src
  
  # Production
  app-prod:
    build:
      context: .
      target: production
```

### Complete Example: Build vs Image

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Using pre-built images
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
  
  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
  
  redis:
    image: redis:7-alpine
  
  # Building custom application
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    image: myapp-backend:latest
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis
  
  # Building with different target
  frontend:
    build:
      context: ./frontend
      target: production
      args:
        REACT_APP_API_URL: http://localhost:3000
    image: myapp-frontend:latest
    ports:
      - "8080:80"
```

### When to Use Build vs Image

**Use Image When:**
- ✅ Using official images (nginx, postgres, redis)
- ✅ Using third-party services
- ✅ Image already built and pushed to registry
- ✅ Quick setup without custom code

**Use Build When:**
- ✅ Custom application code
- ✅ Need to modify base image
- ✅ Development environment
- ✅ Building from source

**Use Both When:**
- ✅ Want to tag built images
- ✅ Planning to push to registry
- ✅ Need image name for other services

### Build Commands

```bash
# Build all services
docker-compose build

# Build specific service
docker-compose build app

# Build without cache
docker-compose build --no-cache

# Build with pull (update base images)
docker-compose build --pull

# Build and start
docker-compose up --build
```

---

## Service Dependency Ordering

### Understanding Dependencies

**Problem**: Services may start before their dependencies are ready.

**Example:**
- Application starts before database is ready
- API starts before cache is initialized
- Frontend starts before backend is available

### depends_on

**Basic Dependency:**
```yaml
services:
  web:
    image: nginx
    depends_on:
      - api
  
  api:
    image: myapi
    depends_on:
      - db
      - cache
  
  db:
    image: postgres
  
  cache:
    image: redis
```

**Startup Order:**
1. db and cache start first (no dependencies)
2. api starts after db and cache
3. web starts after api

**Important**: `depends_on` only waits for container to start, not for service to be ready!

### Dependency with Conditions

**Using Health Checks (Compose v3.8+):**
```yaml
version: '3.8'

services:
  web:
    image: nginx
    depends_on:
      api:
        condition: service_healthy
  
  api:
    image: myapi
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Health Checks

**PostgreSQL:**
```yaml
services:
  postgres:
    image: postgres:14
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
```

**MySQL:**
```yaml
services:
  mysql:
    image: mysql:8
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```

**Redis:**
```yaml
services:
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
```

**Custom Application:**
```yaml
services:
  api:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Wait-for Scripts

**Using wait-for-it.sh:**

**Dockerfile:**
```dockerfile
FROM node:16-alpine

# Add wait-for-it script
ADD https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

WORKDIR /app
COPY . .
RUN npm install

CMD ["npm", "start"]
```

**docker-compose.yml:**
```yaml
services:
  api:
    build: .
    command: /wait-for-it.sh db:5432 -- npm start
    depends_on:
      - db
  
  db:
    image: postgres:14
```

**Using dockerize:**
```yaml
services:
  api:
    build: .
    command: >
      sh -c "dockerize -wait tcp://db:5432 -timeout 60s &&
             npm start"
    depends_on:
      - db
```

### Application-Level Retry Logic

**Node.js Example:**
```javascript
const { Client } = require('pg');

async function connectWithRetry(maxRetries = 5, delay = 5000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const client = new Client({
        host: 'db',
        port: 5432,
        database: 'mydb',
        user: 'postgres',
        password: 'secret'
      });
      
      await client.connect();
      console.log('Connected to database');
      return client;
    } catch (error) {
      console.log(`Connection attempt ${i + 1} failed. Retrying...`);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

connectWithRetry().then(client => {
  // Start application
}).catch(error => {
  console.error('Failed to connect to database:', error);
  process.exit(1);
});
```

### Complete Dependency Example

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Frontend - depends on backend
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      backend:
        condition: service_healthy
    environment:
      - API_URL=http://backend:3000

  # Backend - depends on database and cache
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    environment:
      - DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379

  # Database
  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  postgres-data:
```

### Best Practices

✅ **Always use depends_on for logical dependencies**

✅ **Implement health checks for critical services**

✅ **Add retry logic in application code**

✅ **Use appropriate start_period for slow-starting services**

✅ **Test startup order thoroughly**

✅ **Document dependencies clearly**

❌ **Don't rely solely on depends_on**

❌ **Don't use very short health check intervals**

---

## Summary

**Advanced Docker Compose Features:**

1. **Environment Variables**: Multiple methods for configuration
   - Inline, .env files, substitution
   - Proper precedence and security

2. **Secrets and Configs**: Secure sensitive data
   - Secrets for passwords and keys
   - Configs for configuration files
   - File-based mounting

3. **Build vs Image**: Flexible container sources
   - Image for pre-built containers
   - Build for custom applications
   - Combined for tagging and reuse

4. **Service Dependencies**: Proper startup ordering
   - depends_on for basic ordering
   - Health checks for readiness
   - Application-level retry logic

These features enable building robust, secure, and production-ready multi-container applications.
