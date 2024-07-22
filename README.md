
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
   
   ![Install NPM](path-to-your-image/2-install npm.png)

2. **Add Docker:** Configure Docker under "Manage Jenkins" > "Global Tool Configuration".

3. **Add SonarQube:** Configure SonarQube under "Manage Jenkins" > "Configure System" and provide SonarQube server details.
   
   ![Install Plugins](path-to-your-image/12-install plugins.png)
   
4. **Security Group Configuration:** Ensure your security groups are configured properly in AWS to allow necessary traffic.
   
   ![Configure Security Group](path-to-your-image/1-config sg.png)

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

![Jenkins Pipeline](path-to-your-image/16-working pipeline.png)

### Install Dependencies

Installs Node.js dependencies required for the project.

```groovy
stage('install dependencies') {
    steps {
        sh "npm install"
    }
}
```

![Install Dependencies](path-to-your-image/7-npm i local.png)

### Unit Tests

Runs the unit tests to validate application functionality.

```groovy
stage('Unit tests') {
    steps {
        sh "npm test"
    }
}
```

![Unit Tests](path-to-your-image/6-configure network.png)

### Trivy Filesystem Scan

Scans the filesystem for vulnerabilities.

```groovy
stage('Trivy fs scan') {
    steps {
        sh "trivy fs --format table -o fs-report.html ."
    }
}
```

![Trivy Filesystem Scan](path-to-your-image/10-install sonar qube.png)

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

![SonarQube Analysis](path-to-your-image/11-sonar working.png)

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

![Docker Build & Tag](path-to-your-image/17-docker hub.png)

### Trivy Image Scan

Scans the Docker image for vulnerabilities.

```groovy
stage('Trivy image scan') {
    steps {
        sh "trivy image --format table -o fs-report.html kirox2023/vermegimage:latest"
    }
}
```

![Trivy Image Scan](path-to-your-image/3-configure cloudinary.png)

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

![Docker Push Image](path-to-your-image/18-webhook test jenkins.png)

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

![Docker Deploy to Dev](path-to-your-image/9-configure jenkins.png)

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

![Dockerfile](path-to-your-image/4-mapbox account.png)

### Explanation

- **FROM node:18:** Uses the Node.js 18 image as the base image.
- **WORKDIR /app:** Sets the working directory to `/app` inside the container.
- **COPY package.json package-lock.json ./:** Copies `package.json` and `package-lock.json` to the working directory.
- **RUN npm install:** Installs the Node.js dependencies.
- **COPY . .:** Copies the rest of the application files to the working directory.
- **EXPOSE 3000:** Exposes port 3000 for the application.
- **CMD npm start:** Defines the command to start the application.
