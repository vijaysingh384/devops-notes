# GitHub Actions - Continuous Integration

## Understanding Workflow Automation

GitHub Actions automates software workflows directly in your repository.

### Key Concepts

**Workflow**: Automated process defined in YAML
**Event**: Trigger that starts a workflow
**Job**: Set of steps that execute on same runner
**Step**: Individual task (action or command)
**Action**: Reusable unit of code
**Runner**: Server that runs workflows

## Events and Triggers

### Common Triggers

```yaml
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:  # Manual trigger
```

### Event Types

**push**: Code pushed to repository
**pull_request**: PR opened/updated
**schedule**: Cron-based scheduling
**workflow_dispatch**: Manual trigger
**release**: Release published
**issues**: Issue opened/closed

## Workflow Directory Structure

```
.github/
└── workflows/
    ├── ci.yml
    ├── deploy.yml
    └── test.yml
```

## Key Components

### 1. Workflows

```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
```

### 2. Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."
```

### 3. Steps

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v3
  
  - name: Setup Node.js
    uses: actions/setup-node@v3
    with:
      node-version: '16'
  
  - name: Install dependencies
    run: npm install
  
  - name: Run tests
    run: npm test
```

### 4. Actions

```yaml
- uses: actions/checkout@v3
- uses: actions/setup-java@v3
  with:
    java-version: '11'
    distribution: 'adopt'
```

### 5. Runners

**GitHub-hosted runners:**
- ubuntu-latest
- windows-latest
- macos-latest

**Self-hosted runners:**
```yaml
runs-on: self-hosted
```

## Complete Workflow Example

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
```
