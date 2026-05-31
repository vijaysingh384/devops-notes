# DevOps Course Notes - Complete Guide

## 📚 Course Overview

This repository contains comprehensive notes for Units 3-6 of the DevOps course covering:
- **Unit 3**: Microservices with Docker Compose
- **Unit 4**: Maven Build Automation
- **Unit 5**: Continuous Integration with GitHub Actions
- **Unit 6**: CI/CD with Jenkins

---

## 📂 Repository Structure

```
docker-notes/
├── Unit-3-Microservices-Docker-Compose/
│   ├── 01-Microservices-Architecture.md
│   ├── 02-Docker-Compose-Basics.md
│   ├── 03-Advanced-Docker-Compose.md
│   └── 04-Use-Case-Deployments.md
│
├── Unit-4-Maven-Build-Automation/
│   ├── 01-Maven-Basics.md
│   ├── 02-Build-Lifecycle.md
│   ├── 03-Dependencies.md
│   ├── 04-Maven-Plugins.md
│   └── 05-Maven-Docker-Integration.md
│
├── Unit-5-CI-GitHub-Actions/
│   ├── 01-GitHub-Actions-Basics.md
│   ├── 02-Advanced-GitHub-Actions.md
│   └── 03-Docker-GitHub-Actions.md
│
└── Unit-6-CI-CD-Jenkins/
    ├── 01-Jenkins-Foundations.md
    ├── 02-Jenkins-Pipelines.md
    ├── 03-Docker-Jenkins-Integration.md
    └── 04-Maven-and-CICD.md
```

---

## 📖 Unit 3: Microservices with Docker Compose

### Topics Covered:
- ✅ Microservices Architecture
  - Need for microservices
  - Monolithic vs Microservices
  - Advantages: Scalability, Isolation, Agility
  - API Gateway

- ✅ Docker Compose
  - YAML Structure
  - Writing docker-compose.yml
  - Version, Services, Volumes, Networks
  - Environment Variables
  - Secrets and Configs
  - Build vs Image fields
  - Service Dependency Ordering

- ✅ Use Case Deployments
  - Multi-container applications
  - WordPress + MySQL
  - Node.js + MongoDB
  - Java Spring Boot + PostgreSQL

### Key Files:
1. `01-Microservices-Architecture.md` - Architecture concepts and patterns
2. `02-Docker-Compose-Basics.md` - Docker Compose fundamentals
3. `03-Advanced-Docker-Compose.md` - Advanced features and configurations
4. `04-Use-Case-Deployments.md` - Real-world deployment examples

---

## 📖 Unit 4: Maven Build Automation

### Topics Covered:
- ✅ Maven Basics
  - Why build tools exist
  - Problems solved by automated builds
  - Project Object Model (POM)
  - Directory Structure

- ✅ Build Lifecycle
  - Phases: validate, compile, test, package, verify, install, deploy
  - Clean and Site lifecycles

- ✅ Dependencies
  - Parent POM
  - Dependency Scope
  - Transitive Dependencies
  - Version Conflicts & Resolution
  - Dependency Management

- ✅ Maven Plugins
  - Compiler Plugin
  - Surefire Plugin (unit testing)
  - Shade Plugin (uber jar)
  - Maven Wrapper (mvnw)

- ✅ Maven and Docker Integration
  - dockerfile-maven-plugin
  - Dockerizing Maven applications
  - Pushing artifacts to registries

### Key Files:
1. `01-Maven-Basics.md` - Maven fundamentals and POM
2. `02-Build-Lifecycle.md` - Build phases and lifecycle
3. `03-Dependencies.md` - Dependency management
4. `04-Maven-Plugins.md` - Plugin configuration and usage
5. `05-Maven-Docker-Integration.md` - Docker integration

---

## 📖 Unit 5: Continuous Integration with GitHub Actions

### Topics Covered:
- ✅ Workflow Automation
  - Events and Triggers
  - Workflow Directory Structure
  - Key Components: workflows, jobs, steps, actions, runners

- ✅ Workflow Configuration
  - Triggers: push, pull_request, schedule, workflow_dispatch
  - Jobs & Matrix Strategies
  - Steps & Shell Commands
  - Using Marketplace Actions
  - Language-Specific Actions
  - Caching for Faster Builds

- ✅ Multi-Job Workflows
  - Job Dependencies
  - Parallel Execution
  - Artifact Management

- ✅ Deployments
  - Deploying to servers/cloud
  - GitHub-Hosted Runners
  - Self-Hosted Runners
  - Runner Security & Management

- ✅ Docker & GitHub Actions
  - Building Docker Images in CI
  - Pushing to Docker Hub
  - Pushing to GitHub Container Registry (GHCR)

### Key Files:
1. `01-GitHub-Actions-Basics.md` - Fundamentals and workflow structure
2. `02-Advanced-GitHub-Actions.md` - Advanced patterns and strategies
3. `03-Docker-GitHub-Actions.md` - Docker integration

---

## 📖 Unit 6: CI/CD with Jenkins

### Topics Covered:
- ✅ Jenkins Foundations
  - Architecture (Master/Agent Model)
  - Installation & UI Overview
  - Plugins Management
  - Security, Users, Roles

- ✅ Jenkins Pipelines
  - Freestyle vs Pipeline Jobs
  - Declarative Pipeline Syntax
  - Scripted Pipeline Syntax
  - Jenkinsfile Structure
  - Parameters and Environment Variables
  - Multi-Branch Pipelines

- ✅ Pipeline Stages
  - Checkout Code from Git
  - Build, Test, Package
  - Post Actions
  - Managing Artifacts

- ✅ Docker and Jenkins Integration
  - Building Docker Images
  - Docker Inside Jenkins Agents
  - Using Docker Plugins
  - Publishing Images to Docker Hub/GHCR
  - Jenkins and GitHub Integration

- ✅ Jenkins and Maven
  - Maven Installation in Jenkins
  - Global Tool Configuration
  - Running Maven Builds in Pipelines
  - Code Coverage & Test Reports

- ✅ CI/CD Deployment Flows
  - Triggering Builds (pollSCM, webhook)
  - Pipeline Libraries
  - Jenkins Agents (SSH/Container-based)
  - Deployments to Servers/Clouds
  - Backup & Restore
  - Pipeline Best Practices

### Key Files:
1. `01-Jenkins-Foundations.md` - Jenkins basics and setup
2. `02-Jenkins-Pipelines.md` - Pipeline syntax and structure
3. `03-Docker-Jenkins-Integration.md` - Docker integration
4. `04-Maven-and-CICD.md` - Maven integration and CI/CD flows

---

## 🎯 Learning Objectives

### CO4: Maven Build Automation
Analyze and implement automated builds using Maven through lifecycle management, dependency control, plugins, and Docker integration.

### CO5: GitHub Actions CI
Apply Continuous Integration workflows using GitHub Actions for automated builds, testing, image creation, and deployment.

### CO6: Jenkins CI/CD
Develop end-to-end CI/CD pipelines using Jenkins integrating source control, build tools, containers, and deployments.

---

## 🚀 Quick Start Guide

### For Unit 3 (Docker Compose):
```bash
# Navigate to examples
cd Unit-3-Microservices-Docker-Compose

# Start a sample deployment
docker-compose up -d
```

### For Unit 4 (Maven):
```bash
# Build a Maven project
mvn clean install

# Run with Docker
mvn clean package dockerfile:build
```

### For Unit 5 (GitHub Actions):
```bash
# Create workflow directory
mkdir -p .github/workflows

# Add workflow file
cp Unit-5-CI-GitHub-Actions/examples/ci.yml .github/workflows/
```

### For Unit 6 (Jenkins):
```bash
# Start Jenkins with Docker
docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

# Access Jenkins
open http://localhost:8080
```

---

## 📝 Study Tips

1. **Read in Order**: Follow the unit sequence for progressive learning
2. **Hands-On Practice**: Try all code examples in your environment
3. **Build Projects**: Create sample projects using each technology
4. **Combine Concepts**: Integrate Maven + Docker + GitHub Actions/Jenkins
5. **Review Regularly**: Revisit notes before exams

---

## 🔗 Additional Resources

### Official Documentation:
- [Docker Compose Docs](https://docs.docker.com/compose/)
- [Maven Documentation](https://maven.apache.org/guides/)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Jenkins Documentation](https://www.jenkins.io/doc/)

### Useful Commands Reference:

**Docker Compose:**
```bash
docker-compose up -d
docker-compose down
docker-compose logs -f
docker-compose ps
```

**Maven:**
```bash
mvn clean install
mvn test
mvn package
mvn dependency:tree
```

**GitHub Actions:**
```bash
# Workflows are triggered automatically
# View in: https://github.com/user/repo/actions
```

**Jenkins:**
```bash
# Access via web UI
# http://localhost:8080
```

---

## 📊 Course Credits

- **Lecture**: 2 hours
- **Tutorial**: 0 hours
- **Practical**: 2 hours
- **Total Credits**: 3

---

## ✅ Exam Preparation Checklist

### Unit 3:
- [ ] Understand microservices architecture
- [ ] Write docker-compose.yml files
- [ ] Configure environment variables and secrets
- [ ] Deploy multi-container applications

### Unit 4:
- [ ] Understand Maven lifecycle phases
- [ ] Configure POM files
- [ ] Manage dependencies
- [ ] Use Maven plugins
- [ ] Integrate Maven with Docker

### Unit 5:
- [ ] Create GitHub Actions workflows
- [ ] Configure triggers and events
- [ ] Use matrix strategies
- [ ] Build and push Docker images
- [ ] Deploy applications

### Unit 6:
- [ ] Install and configure Jenkins
- [ ] Write Jenkinsfiles
- [ ] Create pipeline jobs
- [ ] Integrate with Docker and Maven
- [ ] Set up CI/CD pipelines

---

## 📧 Notes

These notes are created for **Session 2025-26** based on the official syllabus.

**Last Updated**: May 31, 2026

---

## 🎓 Good Luck with Your Studies!

Remember: DevOps is about automation, collaboration, and continuous improvement. Practice regularly and build real projects to master these concepts.
