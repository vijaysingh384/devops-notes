# Docker and Jenkins Integration

## Building Docker Images

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("my-app:${env.BUILD_NUMBER}")
                }
            }
        }
    }
}
```

## Docker Inside Jenkins Agents

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8-openjdk-11'
        }
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

## Publishing Images

### Docker Hub

```groovy
stage('Push to Docker Hub') {
    steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                def app = docker.build("username/my-app:${env.BUILD_NUMBER}")
                app.push()
                app.push('latest')
            }
        }
    }
}
```

### GitHub Container Registry

```groovy
stage('Push to GHCR') {
    steps {
        script {
            docker.withRegistry('https://ghcr.io', 'github-token') {
                def app = docker.build("ghcr.io/username/my-app:${env.BUILD_NUMBER}")
                app.push()
            }
        }
    }
}
```

## Jenkins and GitHub Integration

### Webhook Configuration
1. GitHub repo → Settings → Webhooks
2. Add webhook: http://jenkins-url/github-webhook/
3. Select events: Push, Pull Request

### Jenkinsfile with GitHub

```groovy
pipeline {
    agent any
    
    triggers {
        githubPush()
    }
    
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
    }
}
```
