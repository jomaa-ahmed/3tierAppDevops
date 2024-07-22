Certainly! Here’s the refined documentation without the "Additional Configurations" and "Troubleshooting and Maintenance" sections:

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
4. [References](#references)

---

## Pipeline Overview

The Jenkins pipeline automates the process of building, testing, and deploying a 3-tier full stack application. It integrates various tools for continuous integration and delivery (CI/CD), code quality analysis, security scanning, and containerization. The pipeline is designed to ensure code quality, security, and smooth deployment to development environments.

## Jenkins Setup

### Prerequisites

1. **Jenkins Installation:** Ensure Jenkins is installed on an Ubuntu server or any other supported platform.
2. **Plugins:** Install necessary Jenkins plugins such as:
   - Git Plugin
   - NodeJS Plugin
   - Docker Pipeline Plugin
   - SonarQube Scanner Plugin
   - Trivy Plugin (or custom Trivy integration)
3. **Tools Configuration:** Set up tools in Jenkins under "Manage Jenkins" > "Global Tool Configuration".

### Jenkins Configuration

1. **Add NodeJS:** Go to "Manage Jenkins" > "Global Tool Configuration" and add NodeJS with the version `node21`.
2. **Add Docker:** Configure Docker under "Manage Jenkins" > "Global Tool Configuration" and provide Docker installation details.
3. **Add SonarQube:** Configure SonarQube under "Manage Jenkins" > "Configure System" and add SonarQube server details.

## Pipeline Stages

### Git Checkout

This stage fetches the latest code from the Git repository.

```groovy
stage('git checkout') {
    steps {
        git branch: 'main', url: 'https://github.com/jomaa-ahmed/3tierAppDevops.git'
    }
}
```

- **Purpose:** Retrieve the source code for the application from the specified Git branch.
- **Details:** Configures Git to pull the latest changes from the `main` branch of the repository.
- **Commands Used:** `git`

### Install Dependencies

Installs all the necessary Node.js dependencies for the application.

```groovy
stage('install dependencies') {
    steps {
        sh "npm install"
    }
}
```

- **Purpose:** Install project dependencies listed in `package.json`.
- **Details:** Uses npm to ensure all required packages are available for the build and test processes.
- **Commands Used:** `npm install`

### Unit Tests

Runs the unit tests to verify that the application functions as expected.

```groovy
stage('Unit tests') {
    steps {
        sh "npm test"
    }
}
```

- **Purpose:** Execute unit tests to ensure code functionality and correctness.
- **Details:** The test results help in identifying issues early in the development cycle.
- **Commands Used:** `npm test`

### Trivy Filesystem Scan

Scans the application's filesystem for vulnerabilities.

```groovy
stage('Trivy fs scan') {
    steps {
        sh "trivy fs --format table -o fs-report.html ."
    }
}
```

- **Purpose:** Detect vulnerabilities in the filesystem dependencies and configurations.
- **Details:** Generates a report in `fs-report.html` for review.
- **Commands Used:** `trivy fs`

### SonarQube Analysis

Performs static code analysis to identify code smells, bugs, and security vulnerabilities.

```groovy
stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar') {
            sh "\$SCNNAMER_HOME/bin/sonar-scanner -Dsonar.projectKey=myProject -Dsonar.sources=."
        }
    }
}
```

- **Purpose:** Analyze the codebase for quality and security issues.
- **Details:** Utilizes SonarQube to ensure adherence to coding standards and best practices.
- **Commands Used:** `sonar-scanner`

### Quality Gate

(Optional) Ensures that the code meets the quality standards defined in SonarQube before proceeding.

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

- **Purpose:** Validate that the code quality metrics meet the required standards.
- **Details:** Abort the pipeline if quality gate checks fail.
- **Commands Used:** `waitForQualityGate()`

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

- **Purpose:** Create a Docker image and tag it for versioning and deployment.
- **Details:** Uses Docker to build an image from the Dockerfile and tag it with `latest`.
- **Commands Used:** `docker build`

### Trivy Image Scan

Scans the Docker image for vulnerabilities.

```groovy
stage('Trivy image scan') {
    steps {
        sh "trivy image --format table -o fs-report.html kirox2023/vermegimage:latest"
    }
}
```

- **Purpose:** Check the Docker image for known vulnerabilities.
- **Details:** Outputs a report in `fs-report.html`.
- **Commands Used:** `trivy image`

### Docker Push Image

Pushes the Docker image to a Docker registry.

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

- **Purpose:** Upload the built Docker image to a container registry for storage and distribution.
- **Details:** Uses Docker credentials to push the image.
- **Commands Used:** `docker push`

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

- **Purpose:** Run the Docker container in the development environment.
- **Details:** Maps port 3000 on the host to port 3000 in the container.
- **Commands Used:** `docker run`

## References

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Node.js Documentation](https://nodejs.org/en/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/v0.34.2/)

---

This version provides a detailed breakdown of each stage of your pipeline along with the references and setup information needed. If you need more specific details or additional sections, just let me know!