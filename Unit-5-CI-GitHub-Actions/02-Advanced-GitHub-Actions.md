# Advanced GitHub Actions

## Jobs & Matrix Strategies

### Matrix Strategy

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18]
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm test
```

### Job Dependencies

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
  
  deploy:
    needs: [build, test]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

## Steps & Shell Commands

```yaml
steps:
  - name: Multi-line script
    run: |
      echo "Line 1"
      echo "Line 2"
      npm install
      npm test
  
  - name: Conditional step
    if: github.ref == 'refs/heads/main'
    run: echo "On main branch"
  
  - name: Continue on error
    continue-on-error: true
    run: npm run optional-task
```

## Using Marketplace Actions

### Popular Actions

```yaml
# Checkout code
- uses: actions/checkout@v3

# Setup languages
- uses: actions/setup-node@v3
- uses: actions/setup-python@v4
- uses: actions/setup-java@v3

# Upload/Download artifacts
- uses: actions/upload-artifact@v3
  with:
    name: build-output
    path: dist/

- uses: actions/download-artifact@v3
  with:
    name: build-output
```

## Language-Specific Actions

### Node.js

```yaml
- uses: actions/setup-node@v3
  with:
    node-version: '16'
    cache: 'npm'
- run: npm ci
- run: npm test
```

### Python

```yaml
- uses: actions/setup-python@v4
  with:
    python-version: '3.9'
    cache: 'pip'
- run: pip install -r requirements.txt
- run: pytest
```

### Java

```yaml
- uses: actions/setup-java@v3
  with:
    java-version: '11'
    distribution: 'adopt'
    cache: 'maven'
- run: mvn clean install
```

## Using Caching for Faster Builds

### NPM Cache

```yaml
- uses: actions/setup-node@v3
  with:
    node-version: '16'
    cache: 'npm'
```

### Manual Cache

```yaml
- uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

## Multi-Job Workflows

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run lint
  
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
  
  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm run build
```

## Deploying to Servers/Cloud

### SSH Deployment

```yaml
- name: Deploy to server
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.HOST }}
    username: ${{ secrets.USERNAME }}
    key: ${{ secrets.SSH_KEY }}
    script: |
      cd /app
      git pull
      npm install
      pm2 restart app
```

### AWS Deployment

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- name: Deploy to S3
  run: aws s3 sync ./build s3://my-bucket
```
