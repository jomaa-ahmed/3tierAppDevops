
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

![Configure Jenkins](file-Kh5SyEuBwGtIu3tvMnaMBaHG)

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

![Install npm dependencies](file-GWYDpTkeSIGJ4JHlfkMisRSu)

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

![SonarQube Analysis](file-25b26QjPMimvNzbFiQmjp3HL)

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

![Docker Build](file-W4I9NcYfwWGkDKfurjbFp9LX)

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

![Docker Push Image](file-Pl7m40Wz3BFIJyNWWR9R8Fsp)

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

## Setting Up Security Groups

Configure your security group to allow necessary traffic.

![Configure Security Groups](file-QQFIJsGzJe3j2B7ELV4J0Fqh)

## Setting Up API Keys

1. **Cloudinary:** Obtain API keys for media management.
   ![Cloudinary Configuration](file-dyBCggdDEEJYlFvnG9SKSmfo)

2. **Mapbox:** Obtain an API key for mapping services.
   ![Mapbox Account](file-CQXxKge87ECBJRvdB6DdnWbX)

3. **MongoDB Atlas:** Obtain a connection URL for database services.
   ![MongoDB Atlas](file-W4I9NcYfwWGkDKfurjbFp9LX)
  
## Network Configuration

Configure your network settings for optimal performance and security.

![Network Configuration](file-gNbGuV7CnsRqN5agGrH2hVn4)

## Local Execution

For local development and testing, the application is deployed on a local machine using tools like Mobaxterm for SSH connections, NVM for managing Node.js versions, and other necessary configurations.

### Steps for Local Execution

1. **Connect to the Machine:** Use Mobaxterm to connect to your Ubuntu machine using an SSH key.

![Connect to Machine](file-GWYDpTkeSIGJ4JHlfkMisRSu)

2. **Clone the Repository:**
   ```bash
   git clone https://github.com/jomaa-ahmed/3tierAppDevops.git
   cd 3tierAppDevops
   ```

3. **Install Node.js using NVM:**
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
   nvm install 20
   node -v
   npm -v
   ```

![Install Node.js](file-GWYDpTkeSIGJ4JHlfkMisRSu)

4. **Create a `.env` file:**
   ```bash
   vi .env
   ```
   Add the following lines to the file and replace placeholders with your actual values:
   ```env
   CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
   CLOUDINARY_KEY=[Your Cloudinary Key]
   CLOUDINARY_SECRET=[Your Cloudinary Secret]
   MAPBOX_TOKEN=[Your Mapbox Token]
   DB_URL=[

Your MongoDB Atlas Connection URL]
   SECRET=[Your Chosen Secret Key]
   ```

5. **Install project dependencies:**
   ```bash
   npm install
   ```

6. **Start the application:**
   ```bash
   npm start
   ```

7. **Access the app:** Open a web browser and navigate to `http://VM_IP:3000` (replace VM_IP with the IP address of your Ubuntu machine).

![Access the App](file-U61cuPgIQnbF8vCQqgUu7OAQ)

