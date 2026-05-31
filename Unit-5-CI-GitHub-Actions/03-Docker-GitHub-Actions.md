# Docker & GitHub Actions

## Building Docker Images in CI

### Basic Docker Build

```yaml
name: Docker Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t my-app:latest .
    
    - name: Run tests in container
      run: docker run my-app:latest npm test
```

### Multi-Platform Builds

```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v2

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2

- name: Build multi-platform image
  run: |
    docker buildx build       --platform linux/amd64,linux/arm64       -t my-app:latest .
```

## Pushing to Docker Hub

```yaml
name: Build and Push to Docker Hub

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          username/my-app:latest
          username/my-app:${{ github.sha }}
```

## Pushing to GitHub Container Registry (GHCR)

```yaml
name: Build and Push to GHCR

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Login to GHCR
      uses: docker/login-action@v2
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ghcr.io/${{ github.repository }}:latest
          ghcr.io/${{ github.repository }}:${{ github.sha }}
```

## Complete CI/CD with Docker

```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to GHCR
      if: github.event_name != 'pull_request'
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: ${{ github.event_name != 'pull_request' }}
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

## Runners

### GitHub-Hosted Runners

```yaml
jobs:
  build:
    runs-on: ubuntu-latest  # or windows-latest, macos-latest
```

**Specifications:**
- 2-core CPU
- 7 GB RAM
- 14 GB SSD

### Self-Hosted Runners

**Setup:**
1. Go to Settings → Actions → Runners
2. Click "New self-hosted runner"
3. Follow installation instructions

**Usage:**
```yaml
jobs:
  build:
    runs-on: self-hosted
```

### Runner Security & Management

**Best Practices:**
✅ Use GitHub-hosted for public repos
✅ Use self-hosted for private repos with sensitive data
✅ Isolate self-hosted runners
✅ Keep runners updated
✅ Use runner groups for access control
✅ Monitor runner usage

**Runner Labels:**
```yaml
runs-on: [self-hosted, linux, x64, gpu]
```
