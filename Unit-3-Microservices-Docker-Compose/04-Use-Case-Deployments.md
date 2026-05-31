# Use Case Deployments with Docker Compose

## Table of Contents
1. [Multi-Container Applications](#multi-container-applications)
2. [WordPress + MySQL](#wordpress--mysql)
3. [Node.js + MongoDB](#nodejs--mongodb)
4. [Java Spring Boot + PostgreSQL](#java-spring-boot--postgresql)

---

## Multi-Container Applications

### Architecture Overview

**Typical Multi-Container Stack:**
```
┌─────────────────────────────────────┐
│         Load Balancer/Proxy         │
│            (Nginx/Traefik)          │
└──────────────┬──────────────────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────────┐    ┌──────▼──────┐
│  Frontend  │    │   Backend   │
│  (React/   │    │  (Node.js/  │
│   Vue)     │    │   Python)   │
└────────────┘    └──────┬──────┘
                         │
              ┌──────────┴──────────┐
              │                     │
        ┌─────▼─────┐        ┌─────▼─────┐
        │ Database  │        │   Cache   │
        │(Postgres/ │        │  (Redis)  │
        │  MySQL)   │        │           │
        └───────────┘        └───────────┘
```

### Common Patterns

#### 1. Three-Tier Architecture

**Components:**
- **Presentation Tier**: Frontend (React, Vue, Angular)
- **Application Tier**: Backend API (Node.js, Python, Java)
- **Data Tier**: Database (PostgreSQL, MySQL, MongoDB)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Presentation Tier
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - frontend-network

  # Application Tier
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - database
    networks:
      - frontend-network
      - backend-network

  # Data Tier
  database:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend-network

volumes:
  db-data:

networks:
  frontend-network:
  backend-network:
```

#### 2. Microservices with API Gateway

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - product-service
      - order-service

  # User Service
  user-service:
    build: ./services/user
    environment:
      - DB_HOST=user-db
    depends_on:
      - user-db

  user-db:
    image: postgres:14
    environment:
      POSTGRES_DB: users

  # Product Service
  product-service:
    build: ./services/product
    environment:
      - DB_HOST=product-db
    depends_on:
      - product-db

  product-db:
    image: postgres:14
    environment:
      POSTGRES_DB: products

  # Order Service
  order-service:
    build: ./services/order
    environment:
      - DB_HOST=order-db
    depends_on:
      - order-db

  order-db:
    image: postgres:14
    environment:
      POSTGRES_DB: orders
```

#### 3. Full Stack with Caching

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - frontend
      - backend

  frontend:
    build: ./frontend
    expose:
      - "3000"

  backend:
    build: ./backend
    expose:
      - "5000"
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

---

## WordPress + MySQL

### Overview

**Components:**
- WordPress (PHP application)
- MySQL (Database)
- Optional: phpMyAdmin (Database management)

### Basic WordPress Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db
    restart: always

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db-data:/var/lib/mysql
    restart: always

volumes:
  wordpress-data:
  db-data:
```

**Usage:**
```bash
# Start services
docker-compose up -d

# Access WordPress
# Open browser: http://localhost:8080

# Stop services
docker-compose down

# Remove everything including volumes
docker-compose down -v
```

### WordPress with phpMyAdmin

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      db:
        condition: service_healthy
    restart: always

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  phpmyadmin:
    image: phpmyadmin:latest
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: root_password
    depends_on:
      - db
    restart: always

volumes:
  wordpress-data:
  db-data:
```

**Access Points:**
- WordPress: http://localhost:8080
- phpMyAdmin: http://localhost:8081

### Production WordPress Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - wordpress-data:/var/www/html
    depends_on:
      - wordpress
    restart: always

  wordpress:
    image: wordpress:php8.1-fpm
    expose:
      - "9000"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}
      WORDPRESS_CONFIG_EXTRA: |
        define('WP_REDIS_HOST', 'redis');
        define('WP_CACHE', true);
    volumes:
      - wordpress-data:/var/www/html
      - ./php.ini:/usr/local/etc/php/conf.d/custom.ini
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: always

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
      - ./mysql.cnf:/etc/mysql/conf.d/custom.cnf
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: always

  backup:
    image: mysql:8.0
    volumes:
      - ./backups:/backups
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    entrypoint: |
      bash -c 'bash -s <<EOF
      trap "break;exit" SIGHUP SIGINT SIGTERM
      sleep 300
      while /bin/true; do
        mysqldump -h db -u root -p${DB_ROOT_PASSWORD} --all-databases | gzip -c > /backups/backup_\$(date +%Y%m%d_%H%M%S).sql.gz
        find /backups -type f -mtime +7 -delete
        sleep 86400
      done
      EOF'
    depends_on:
      - db

volumes:
  wordpress-data:
  db-data:
  redis-data:
```

**.env file:**
```bash
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=secure_password_here
DB_ROOT_PASSWORD=secure_root_password_here
```

---

## Node.js + MongoDB

### Overview

**Components:**
- Node.js application (Express API)
- MongoDB (NoSQL database)
- Optional: Mongo Express (Database UI)

### Basic Node.js + MongoDB Setup

**Project Structure:**
```
project/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── .env
```

**backend/Dockerfile:**
```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

**backend/package.json:**
```json
{
  "name": "nodejs-mongodb-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.0"
  }
}
```

**backend/server.js:**
```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

// MongoDB connection with retry logic
const connectDB = async () => {
  const maxRetries = 5;
  for (let i = 0; i < maxRetries; i++) {
    try {
      await mongoose.connect(process.env.MONGODB_URI, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
      });
      console.log('MongoDB connected successfully');
      return;
    } catch (error) {
      console.log(`Connection attempt ${i + 1} failed. Retrying...`);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
};

// Define schema and model
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.get('/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/users', async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Start server
const PORT = process.env.PORT || 3000;

connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}).catch(error => {
  console.error('Failed to connect to MongoDB:', error);
  process.exit(1);
});
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - PORT=3000
    depends_on:
      mongo:
        condition: service_healthy
    restart: unless-stopped

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
      MONGO_INITDB_DATABASE: myapp
    volumes:
      - mongo-data:/data/db
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  mongo-data:
```

### Node.js + MongoDB with Mongo Express

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - MONGODB_URI=mongodb://admin:password@mongo:27017/myapp?authSource=admin
      - PORT=3000
    depends_on:
      mongo:
        condition: service_healthy
    volumes:
      - ./backend:/app
      - /app/node_modules
    restart: unless-stopped

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
      MONGO_INITDB_DATABASE: myapp
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  mongo-express:
    image: mongo-express:latest
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: password
      ME_CONFIG_MONGODB_URL: mongodb://admin:password@mongo:27017/
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: admin
    depends_on:
      - mongo
    restart: unless-stopped

volumes:
  mongo-data:
```

**Access Points:**
- API: http://localhost:3000
- Mongo Express: http://localhost:8081 (admin/admin)

### Production Node.js + MongoDB

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - backend
    restart: always

  backend:
    build:
      context: ./backend
      target: production
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  mongo:
    image: mongo:6
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
    volumes:
      - mongo-data:/data/db
      - ./mongod.conf:/etc/mongod.conf
    command: mongod --config /etc/mongod.conf
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    restart: always

volumes:
  mongo-data:
  redis-data:
```

---

## Java Spring Boot + PostgreSQL

### Overview

**Components:**
- Spring Boot application (Java)
- PostgreSQL (Relational database)
- Optional: pgAdmin (Database management)

### Project Structure

```
project/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
│       └── main/
│           ├── java/
│           └── resources/
│               └── application.properties
└── .env
```

### Spring Boot Application

**backend/Dockerfile:**
```dockerfile
# Multi-stage build
FROM maven:3.8-openjdk-17 AS build

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Production stage
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copy jar from build stage
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**backend/pom.xml:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**backend/src/main/resources/application.properties:**
```properties
# Server
server.port=8080

# Database
spring.datasource.url=jdbc:postgresql://postgres:5432/mydb
spring.datasource.username=${DB_USER:postgres}
spring.datasource.password=${DB_PASSWORD:password}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true

# Actuator
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=always
```

### Basic Spring Boot + PostgreSQL Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/mydb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  postgres-data:
```

### Spring Boot + PostgreSQL with pgAdmin

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/mydb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./backend/src:/app/src
    restart: unless-stopped

  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  pgadmin:
    image: dpage/pgadmin4:latest
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    depends_on:
      - postgres
    restart: unless-stopped

volumes:
  postgres-data:
  pgadmin-data:
```

**Access Points:**
- Spring Boot API: http://localhost:8080
- Health Check: http://localhost:8080/actuator/health
- pgAdmin: http://localhost:5050 (admin@admin.com/admin)

### Production Spring Boot + PostgreSQL

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - backend
    restart: always

  backend:
    build:
      context: ./backend
      args:
        - MAVEN_OPTS=-Xmx512m
    expose:
      - "8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/${DB_NAME}
      - SPRING_DATASOURCE_USERNAME=${DB_USER}
      - SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}
      - JAVA_OPTS=-Xmx512m -Xms256m
    depends_on:
      postgres:
        condition: service_healthy
    restart: always
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./postgresql.conf:/etc/postgresql/postgresql.conf
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  backup:
    image: postgres:15
    volumes:
      - ./backups:/backups
      - postgres-data:/var/lib/postgresql/data
    environment:
      PGPASSWORD: ${DB_PASSWORD}
    entrypoint: |
      bash -c 'bash -s <<EOF
      trap "break;exit" SIGHUP SIGINT SIGTERM
      sleep 300
      while /bin/true; do
        pg_dump -h postgres -U ${DB_USER} ${DB_NAME} | gzip > /backups/backup_\$(date +%Y%m%d_%H%M%S).sql.gz
        find /backups -type f -mtime +7 -delete
        sleep 86400
      done
      EOF'
    depends_on:
      - postgres

volumes:
  postgres-data:
```

**.env file:**
```bash
DB_NAME=mydb
DB_USER=appuser
DB_PASSWORD=secure_password_here
```

### Complete Full-Stack Example

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    restart: unless-stopped

  # Backend
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/mydb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
      - SPRING_REDIS_HOST=redis
      - SPRING_REDIS_PORT=6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  # Database
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Cache
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: unless-stopped

  # Database Management
  pgadmin:
    image: dpage/pgadmin4:latest
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    depends_on:
      - postgres
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
  pgadmin-data:
```

---

## Summary

**Multi-Container Deployment Patterns:**

1. **WordPress + MySQL**: Content management system
   - Quick setup for blogs and websites
   - Persistent data with volumes
   - Optional phpMyAdmin for database management

2. **Node.js + MongoDB**: Modern web applications
   - NoSQL database for flexible schemas
   - Mongo Express for database visualization
   - Suitable for real-time applications

3. **Java Spring Boot + PostgreSQL**: Enterprise applications
   - Robust relational database
   - Maven-based builds
   - pgAdmin for database administration

**Common Best Practices:**
- Use health checks for dependencies
- Implement proper volume management
- Configure environment variables securely
- Add database management tools for development
- Use multi-stage builds for optimization
- Implement backup strategies for production

These patterns provide foundation for deploying real-world applications with Docker Compose.
