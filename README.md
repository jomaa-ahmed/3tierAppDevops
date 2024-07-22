Certainly! Here’s the updated documentation integrating the Dockerfile and providing detailed instructions on its role in the DevOps pipeline.

---

# DevOps Pipeline Documentation

## Table of Contents
1. [Pipeline Overview](#pipeline-overview)
2. [Jenkins Setup](#jenkins-setup)
3. [Pipeline Stages](#pipeline-stages)
   - [Git Checkout](#git-checkout)
   - [Install Dependencies](#install-dependencies)
   - [Unit Tests](#unit-tests)
   - [Trivy Filesystem Scan](#trivy-filesystem-scan)
   - [SonarQube Analysis](#sonarqube-analysis)
   - [Quality Gate](#quality-gate)
   - [Docker Build & Tag](#docker-build--tag)
   - [Trivy Image Scan](#trivy-image-scan)
   - [Docker Push Image](#docker-push-image)
   - [Docker Deploy to Dev](#docker-deploy-to-dev)
4. [Dockerfile](#dockerfile)


---

## Pipeline Overview

The Jenkins pipeline automates the continuous integration and deployment process for a 3-tier full stack application. It integrates various tools for building, testing, analyzing, and deploying the application. This pipeline ensures high code quality and security, and it leverages Docker for containerization.

## Jenkins Setup

### Prerequisites

1. **Jenkins Installation:** Ensure Jenkins is installed on your server.
2. **Plugins:** Install required Jenkins plugins such as:
   - Git Plugin
   - NodeJS Plugin
   - Docker Pipeline Plugin
   - SonarQube Scanner Plugin
   - Trivy Plugin (or custom Trivy integration)
3. **Tools Configuration:** Configure NodeJS, Docker, and SonarQube in Jenkins.

### Jenkins Configuration

1. **Add NodeJS:** Go to "Manage Jenkins" > "Global Tool Configuration" and add NodeJS with the version `node21`.
2. **Add Docker:** Configure Docker under "Manage Jenkins" > "Global Tool Configuration".
3. **Add SonarQube:** Configure SonarQube under "Manage Jenkins" > "Configure System" and provide SonarQube server details.

## Pipeline Stages

### Git Checkout

Retrieves the latest code from the Git repository.

```groovy
stage('git checkout') {
    steps {
        git branch: 'main', url: 'https://github.com/jomaa-ahmed/3tierAppDevops.git'
    }
}
```

### Install Dependencies

Installs Node.js dependencies required for the project.

```groovy
stage('install dependencies') {
    steps {
        sh "npm install"
    }
}
```

### Unit Tests

Runs the unit tests to validate application functionality.

```groovy
stage('Unit tests') {
    steps {
        sh "npm test"
    }
}
```

### Trivy Filesystem Scan

Scans the filesystem for vulnerabilities.

```groovy
stage('Trivy fs scan') {
    steps {
        sh "trivy fs --format table -o fs-report.html ."
    }
}
```

### SonarQube Analysis

Performs static code analysis to detect issues in the codebase.

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar') {
            sh "\$SCNNAMER_HOME/bin/sonar-scanner -Dsonar.projectKey=myProject -Dsonar.sources=."
        }
    }
}
```

### Quality Gate

(Optional) Ensures the code quality meets the standards set in SonarQube.

```groovy
stage('Quality Gate') {
    steps {
        script {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    }
}
```

### Docker Build & Tag

Builds and tags the Docker image for the application.

```groovy
stage('docker build & Tag') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                sh "docker build -t kirox2023/vermegimage:latest ."
            }
        }
    }
}
```

### Trivy Image Scan

Scans the Docker image for vulnerabilities.

```groovy
stage('Trivy image scan') {
    steps {
        sh "trivy image --format table -o fs-report.html kirox2023/vermegimage:latest"
    }
}
```

### Docker Push Image

Pushes the Docker image to a container registry.

```groovy
stage('docker Push image') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                sh "docker push kirox2023/vermegimage:latest"
            }
        }
    }
}
```

### Docker Deploy to Dev

Deploys the Docker image to the development environment.

```groovy
stage('docker deploy to dev') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                sh "docker run -d -p 3000:3000 kirox2023/vermegimage:latest"
            }
        }
    }
}
```

## Dockerfile

The Dockerfile defines the steps to build a Docker image for the application. Here’s an overview of its contents:

```dockerfile
# Use Node 18 as parent image
FROM node:18

# Change the working directory on the Docker image to /app
WORKDIR /app

# Copy package.json and package-lock.json to the /app directory
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy the rest of project files into this image
COPY . .

# Expose application port
EXPOSE 3000

# Start the application
CMD npm start
```

### Explanation

- **FROM node:18:** Uses the Node.js 18 image as the base image.
- **WORKDIR /app:** Sets the working directory to `/app` inside the container.
- **COPY package.json package-lock.json ./:** Copies `package.json` and `package-lock.json` to the working directory.
- **RUN npm install:** Installs the Node.js dependencies.
- **COPY . .:** Copies the rest of the application files to the working directory.
- **EXPOSE 3000:** Exposes port 3000 for the application.
- **CMD npm start:** Defines the command to start the application.



