# Jenkins Pipelines

## Freestyle vs Pipeline Jobs

**Freestyle:**
- GUI-based configuration
- Limited flexibility
- Good for simple tasks

**Pipeline:**
- Code-based (Jenkinsfile)
- Version controlled
- Highly flexible

## Declarative Pipeline Syntax

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

## Scripted Pipeline Syntax

```groovy
node {
    stage('Build') {
        echo 'Building...'
    }
    stage('Test') {
        echo 'Testing...'
    }
}
```

## Jenkinsfile Structure

```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'my-app'
    }
    
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.0')
        choice(name: 'ENV', choices: ['dev', 'prod'])
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    
    post {
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

## Multi-Branch Pipelines

```groovy
pipeline {
    agent any
    
    stages {
        stage('Deploy to Dev') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to dev'
            }
        }
        
        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production'
            }
        }
    }
}
```
