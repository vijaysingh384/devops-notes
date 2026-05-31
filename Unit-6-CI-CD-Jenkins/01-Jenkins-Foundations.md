# Jenkins Foundations

## Jenkins Architecture - Master/Agent Model

**Master (Controller):**
- Schedules builds
- Monitors agents
- Records build results
- Serves Jenkins UI

**Agent (Node):**
- Executes build jobs
- Can be on different machines
- Can have different environments

## Installation

### Docker Installation

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Get Initial Admin Password

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Access Jenkins
Open browser: http://localhost:8080

## UI Overview

**Main Components:**
- Dashboard: Overview of all jobs
- New Item: Create new jobs
- People: User management
- Build History: Past builds
- Manage Jenkins: Configuration

## Plugins Management

### Essential Plugins
- Git Plugin
- Pipeline Plugin
- Docker Plugin
- Blue Ocean (modern UI)
- Credentials Plugin

## Security, Users, Roles

### Enable Security
Manage Jenkins → Configure Global Security

### Authorization Strategies
- Matrix-based security
- Project-based Matrix Authorization
- Role-Based Access Control

### Creating Users
1. Manage Jenkins → Manage Users
2. Create User
3. Fill details and save
