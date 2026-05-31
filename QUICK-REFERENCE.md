# Quick Reference Guide - DevOps Course

## 🚀 Unit 3: Docker Compose Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# List services
docker-compose ps

# Scale services
docker-compose up --scale service=3

# Rebuild images
docker-compose build

# Remove volumes
docker-compose down -v
```

## 🔨 Unit 4: Maven Commands

```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Package application
mvn package

# Install to local repo
mvn install

# Skip tests
mvn install -DskipTests

# View dependency tree
mvn dependency:tree

# Run application
mvn exec:java

# Build with Docker plugin
mvn clean package dockerfile:build

# Push to registry
mvn dockerfile:push
```

## ⚡ Unit 5: GitHub Actions Workflow Template

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '16'
      - run: npm install
      - run: npm test
      - run: npm run build
```

## 🔧 Unit 6: Jenkins Pipeline Template

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t my-app .'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker push my-app'
            }
        }
    }
}
```

## 📝 Common Docker Commands

```bash
# Build image
docker build -t image-name .

# Run container
docker run -d -p 8080:8080 image-name

# List containers
docker ps

# Stop container
docker stop container-id

# Remove container
docker rm container-id

# View logs
docker logs container-id

# Execute command in container
docker exec -it container-id bash
```

## 🎯 Exam Quick Tips

### Unit 3 - Docker Compose
- Know YAML syntax
- Understand service dependencies
- Remember volume types (named, bind, anonymous)
- Network isolation concepts

### Unit 4 - Maven
- Memorize lifecycle phases order
- Understand dependency scopes
- Know common plugins (compiler, surefire, shade)
- POM structure (GAV coordinates)

### Unit 5 - GitHub Actions
- Workflow triggers (push, pull_request, schedule)
- Matrix strategy syntax
- Secrets usage
- Docker build and push actions

### Unit 6 - Jenkins
- Declarative vs Scripted pipeline
- Jenkinsfile structure
- Agent types (any, docker, label)
- Post actions (success, failure, always)

## 📊 Key Concepts Summary

### Microservices Advantages
1. **Scalability** - Scale services independently
2. **Isolation** - Fault tolerance
3. **Agility** - Faster development
4. **Technology Flexibility** - Use different stacks

### Maven Lifecycle Phases
1. validate → 2. compile → 3. test → 4. package → 5. verify → 6. install → 7. deploy

### Dependency Scopes
- **compile** - Default, all classpaths
- **provided** - Provided by runtime
- **runtime** - Not needed for compilation
- **test** - Only for testing
- **system** - Explicit path
- **import** - Import BOM

### CI/CD Pipeline Stages
1. **Checkout** - Get source code
2. **Build** - Compile application
3. **Test** - Run tests
4. **Package** - Create artifacts
5. **Deploy** - Deploy to environment

## 🔐 Security Best Practices

### Docker Compose
```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Maven
```xml
<!-- Use settings.xml for credentials -->
<servers>
  <server>
    <id>docker.io</id>
    <username>${env.DOCKER_USER}</username>
    <password>${env.DOCKER_PASS}</password>
  </server>
</servers>
```

### GitHub Actions
```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Jenkins
```groovy
withCredentials([usernamePassword(
    credentialsId: 'docker-hub',
    usernameVariable: 'USER',
    passwordVariable: 'PASS'
)]) {
    sh 'docker login -u $USER -p $PASS'
}
```

## 📚 Important File Locations

```
Docker Compose:    ./docker-compose.yml
Maven POM:         ./pom.xml
GitHub Actions:    ./.github/workflows/*.yml
Jenkins:           ./Jenkinsfile
Dockerfile:        ./Dockerfile
```

## ⚙️ Configuration Examples

### docker-compose.yml
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  db:
    image: postgres:14
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

### pom.xml
```xml
<project>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0.0</version>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## 🎓 Study Checklist

- [ ] Understand microservices architecture
- [ ] Write docker-compose files
- [ ] Configure Maven POM
- [ ] Know Maven lifecycle phases
- [ ] Create GitHub Actions workflows
- [ ] Write Jenkinsfiles
- [ ] Integrate Docker with CI/CD
- [ ] Understand dependency management
- [ ] Know security best practices
- [ ] Practice hands-on examples

---

**Remember**: Practice is key! Try building a complete project using all these technologies together.
