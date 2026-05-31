# Jenkins Maven Integration & CI/CD

## Maven Installation in Jenkins

### Global Tool Configuration
1. Manage Jenkins → Global Tool Configuration
2. Maven → Add Maven
3. Name: Maven-3.8
4. Install automatically
5. Save

## Running Maven Builds

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8'
        jdk 'JDK-11'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

## Code Coverage & Test Reports

```groovy
stage('Test') {
    steps {
        sh 'mvn test'
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
            jacoco()
        }
    }
}
```

## Triggering Builds

### pollSCM

```groovy
triggers {
    pollSCM('H/5 * * * *')  // Poll every 5 minutes
}
```

### Webhook

```groovy
triggers {
    githubPush()
}
```

## Pipeline Libraries

**Shared Library Structure:**
```
jenkins-library/
├── vars/
│   └── buildApp.groovy
└── src/
```

**Using Shared Library:**
```groovy
@Library('jenkins-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp('clean install')
            }
        }
    }
}
```

## Jenkins Agents

### SSH Agent

```groovy
agent {
    label 'linux-agent'
}
```

### Docker Agent

```groovy
agent {
    docker {
        image 'maven:3.8-openjdk-11'
    }
}
```

## Deployments

### SSH Deployment

```groovy
stage('Deploy') {
    steps {
        sshagent(['ssh-credentials']) {
            sh 'scp target/*.jar user@server:/opt/app/'
            sh 'ssh user@server "systemctl restart myapp"'
        }
    }
}
```

### Kubernetes Deployment

```groovy
stage('Deploy to Kubernetes') {
    steps {
        script {
            kubernetesDeploy(
                configs: 'k8s/deployment.yaml',
                kubeconfigId: 'kubeconfig'
            )
        }
    }
}
```

## Backup & Restore

### Backup Jenkins

```bash
tar -czf jenkins-backup.tar.gz /var/jenkins_home
```

### Restore Jenkins

```bash
tar -xzf jenkins-backup.tar.gz -C /var/jenkins_home
```

## Pipeline Best Practices

✅ Use Declarative Pipeline
✅ Version control Jenkinsfile
✅ Use shared libraries
✅ Implement error handling
✅ Use credentials securely
✅ Clean workspace after build
✅ Parallel execution when possible
✅ Proper logging
✅ Test pipelines in non-production first
