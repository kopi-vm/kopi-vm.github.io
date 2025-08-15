# CI/CD Integration

Integrate Kopi into your continuous integration and deployment pipelines.

## GitHub Actions

### Basic Setup

```yaml
name: Build and Test
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Kopi
        run: |
          curl -fsSL https://kopi-vm.github.io/install.sh | bash
          echo "$HOME/.kopi/shims" >> $GITHUB_PATH
      
      - name: Setup JDK
        run: |
          kopi install $(cat .kopi-version)
      
      - name: Build
        run: ./gradlew build
      
      - name: Test
        run: ./gradlew test
```

### Matrix Builds

Test with multiple JDK versions:

```yaml
name: Matrix Build
on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        jdk: [11, 17, 21]
        distribution: [temurin, corretto]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Kopi (Unix)
        if: runner.os != 'Windows'
        run: |
          curl -fsSL https://kopi-vm.github.io/install.sh | bash
          echo "$HOME/.kopi/shims" >> $GITHUB_PATH
      
      - name: Install Kopi (Windows)
        if: runner.os == 'Windows'
        run: |
          iwr -useb https://kopi-vm.github.io/install.ps1 | iex
          echo "$env:USERPROFILE\.kopi\shims" >> $env:GITHUB_PATH
      
      - name: Setup JDK
        run: kopi install ${{ matrix.distribution }}@${{ matrix.jdk }}
      
      - name: Build and Test
        run: |
          java --version
          ./gradlew build test
```

### Caching

Speed up builds with caching:

```yaml
- name: Cache Kopi JDKs
  uses: actions/cache@v3
  with:
    path: ~/.kopi/jdks
    key: ${{ runner.os }}-kopi-${{ hashFiles('.kopi-version') }}
    restore-keys: |
      ${{ runner.os }}-kopi-

- name: Cache Kopi Metadata
  uses: actions/cache@v3
  with:
    path: ~/.kopi/cache
    key: ${{ runner.os }}-kopi-metadata-${{ hashFiles('.kopi-version') }}
```

## GitLab CI

### Basic Pipeline

```yaml
image: ubuntu:latest

variables:
  KOPI_HOME: "$CI_PROJECT_DIR/.kopi"
  PATH: "$KOPI_HOME/shims:$PATH"

before_script:
  - apt-get update && apt-get install -y curl
  - curl -fsSL https://kopi-vm.github.io/install.sh | bash
  - kopi install $(cat .kopi-version)

build:
  stage: build
  script:
    - ./gradlew build
  artifacts:
    paths:
      - build/

test:
  stage: test
  script:
    - ./gradlew test
  dependencies:
    - build
```

### Docker Integration

```yaml
build-docker:
  image: docker:latest
  services:
    - docker:dind
  script:
    - |
      cat > Dockerfile << EOF
      FROM ubuntu:22.04
      RUN apt-get update && apt-get install -y curl
      RUN curl -fsSL https://kopi-vm.github.io/install.sh | bash
      ENV PATH="/root/.kopi/shims:\$PATH"
      COPY .kopi-version .
      RUN kopi install \$(cat .kopi-version)
      COPY . /app
      WORKDIR /app
      RUN ./gradlew build
      CMD ["java", "-jar", "build/libs/app.jar"]
      EOF
    - docker build -t myapp:$CI_COMMIT_SHA .
    - docker push myapp:$CI_COMMIT_SHA
```

## Jenkins

### Jenkinsfile

```groovy
pipeline {
    agent any
    
    environment {
        PATH = "${env.HOME}/.kopi/shims:${env.PATH}"
    }
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    curl -fsSL https://kopi-vm.github.io/install.sh | bash
                    kopi install $(cat .kopi-version)
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh './gradlew build'
            }
        }
        
        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './gradlew deploy'
            }
        }
    }
}
```

### Shared Library

Create a Jenkins shared library:

```groovy
// vars/setupKopi.groovy
def call(String version = null) {
    sh '''
        if ! command -v kopi &> /dev/null; then
            curl -fsSL https://kopi-vm.github.io/install.sh | bash
        fi
    '''
    
    if (version) {
        sh "kopi install ${version}"
    } else if (fileExists('.kopi-version')) {
        sh 'kopi install $(cat .kopi-version)'
    }
}
```

Usage:

```groovy
@Library('kopi-utils') _

pipeline {
    agent any
    stages {
        stage('Setup') {
            steps {
                setupKopi('temurin@21')
            }
        }
    }
}
```

## CircleCI

```yaml
version: 2.1

executors:
  java-executor:
    docker:
      - image: cimg/base:stable
    environment:
      PATH: /home/circleci/.kopi/shims:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

jobs:
  build:
    executor: java-executor
    steps:
      - checkout
      
      - restore_cache:
          keys:
            - kopi-{{ checksum ".kopi-version" }}
            - kopi-
      
      - run:
          name: Install Kopi
          command: |
            curl -fsSL https://kopi-vm.github.io/install.sh | bash
            kopi install $(cat .kopi-version)
      
      - save_cache:
          key: kopi-{{ checksum ".kopi-version" }}
          paths:
            - ~/.kopi
      
      - run:
          name: Build
          command: ./gradlew build
      
      - run:
          name: Test
          command: ./gradlew test

workflows:
  main:
    jobs:
      - build
```

## Travis CI

```yaml
language: minimal
os: linux
dist: focal

cache:
  directories:
    - $HOME/.kopi

before_install:
  - curl -fsSL https://kopi-vm.github.io/install.sh | bash
  - export PATH="$HOME/.kopi/shims:$PATH"

install:
  - kopi install $(cat .kopi-version)

script:
  - java --version
  - ./gradlew build
  - ./gradlew test

matrix:
  include:
    - env: JDK=11
      install: kopi install temurin@11
    - env: JDK=17
      install: kopi install temurin@17
    - env: JDK=21
      install: kopi install temurin@21
```

## Docker

### Dockerfile

```dockerfile
# Multi-stage build with Kopi
FROM ubuntu:22.04 AS builder

# Install dependencies
RUN apt-get update && apt-get install -y curl

# Install Kopi
RUN curl -fsSL https://kopi-vm.github.io/install.sh | bash
ENV PATH="/root/.kopi/shims:$PATH"

# Copy version file and install JDK
WORKDIR /app
COPY .kopi-version .
RUN kopi install $(cat .kopi-version)

# Copy source and build
COPY . .
RUN ./gradlew build

# Runtime stage
FROM ubuntu:22.04

# Install Kopi for runtime
RUN apt-get update && apt-get install -y curl && \
    curl -fsSL https://kopi-vm.github.io/install.sh | bash

ENV PATH="/root/.kopi/shims:$PATH"

# Copy JDK from builder
COPY --from=builder /root/.kopi /root/.kopi

# Copy application
COPY --from=builder /app/build/libs/*.jar /app/app.jar

WORKDIR /app
CMD ["java", "-jar", "app.jar"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  app:
    build: .
    environment:
      - KOPI_VERSION=21
    volumes:
      - ~/.kopi:/root/.kopi:cached
    ports:
      - "8080:8080"
```

## Kubernetes

### ConfigMap for Version

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: java-version
data:
  version: "temurin@21"
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
spec:
  replicas: 3
  template:
    spec:
      initContainers:
      - name: setup-jdk
        image: ubuntu:22.04
        command:
          - sh
          - -c
          - |
            apt-get update && apt-get install -y curl
            curl -fsSL https://kopi-vm.github.io/install.sh | bash
            kopi install $(cat /config/version)
        volumeMounts:
        - name: kopi-home
          mountPath: /root/.kopi
        - name: config
          mountPath: /config
      
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: PATH
          value: "/root/.kopi/shims:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        volumeMounts:
        - name: kopi-home
          mountPath: /root/.kopi
      
      volumes:
      - name: kopi-home
        emptyDir: {}
      - name: config
        configMap:
          name: java-version
```

## Best Practices

### Version Pinning

Always specify exact versions in CI/CD:

```yaml
# Good - reproducible
kopi install temurin@21.0.2+13.0.LTS

# Avoid - may change
kopi install 21
```

### Caching Strategy

Cache these directories:
- `~/.kopi/jdks` - Installed JDKs (large, changes rarely)
- `~/.kopi/cache` - Metadata cache (small, changes frequently)
- `~/.kopi/shims` - Shim scripts (tiny, changes with Kopi version)

### Error Handling

```bash
#!/bin/bash
set -euo pipefail

# Install with error checking
if ! kopi install $(cat .kopi-version); then
    echo "Failed to install JDK, trying fallback"
    kopi install temurin@21
fi

# Verify installation
java --version || exit 1
```

### Security

1. **Verify downloads** - Kopi verifies checksums automatically
2. **Use specific versions** - Avoid latest/nightly builds
3. **Scan images** - Include security scanning in pipeline
4. **Minimal containers** - Use distroless or Alpine for runtime

## Performance Optimization

### Parallel Installation

```yaml
# Install multiple JDKs in parallel
- name: Install JDKs
  run: |
    kopi install temurin@17 &
    kopi install temurin@21 &
    wait
```

### Pre-built Images

Create base images with Kopi pre-installed:

```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl && \
    curl -fsSL https://kopi-vm.github.io/install.sh | bash && \
    kopi install temurin@21 && \
    kopi install temurin@17
```

## Troubleshooting

### Installation Failures

```yaml
- name: Debug Kopi Installation
  if: failure()
  run: |
    kopi --version
    kopi doctor
    kopi list
    ls -la ~/.kopi/
```

### Path Issues

```yaml
- name: Fix PATH
  run: |
    echo "PATH=$HOME/.kopi/shims:$PATH" >> $GITHUB_ENV
    echo "$HOME/.kopi/shims" >> $GITHUB_PATH
```

## Next Steps

- [Working with Projects](projects.md) - Project configuration
- [Shell Integration](shell-integration.md) - Local development
- [Migration Guide](migration.md) - Migrating from other tools