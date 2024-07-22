Here's the complete documentation in Markdown format:

```markdown
# 3-Tier Full Stack Project Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Setting Up Jenkins](#setting-up-jenkins)
4. [Installing Docker](#installing-docker)
5. [Setting Up SonarQube](#setting-up-sonarqube)
6. [Local Execution](#local-execution)
7. [Dev Deployment Pipeline](#dev-deployment-pipeline)
8. [Production Deployment Pipeline](#production-deployment-pipeline)
9. [Additional Configurations](#additional-configurations)

## Introduction
This documentation provides a comprehensive guide for setting up and deploying a 3-tier full stack project using various tools and platforms. The project integrates Jenkins for continuous integration and delivery, Docker for containerization, SonarQube for code quality analysis, Cloudinary for media management, Mapbox for mapping services, and MongoDB Atlas for database services.

## Project Overview
The 3-tier architecture consists of:
- **Presentation Tier**: This is the user interface layer that interacts with users.
- **Logic Tier**: This layer processes user inputs and makes logical decisions.
- **Data Tier**: This layer handles data storage and management.

The project aims to demonstrate the integration of these tiers using modern DevOps practices and tools.

## Setting Up Jenkins
Jenkins is an open-source automation server used to automate parts of the software development process, such as building, testing, and deploying code. In this project, Jenkins is used for continuous integration and continuous delivery (CI/CD). The setup involves installing Jenkins on an Ubuntu server, configuring it for use, and ensuring it can interact with other components of the project.

### Jenkins Installation Script
```bash
#!/bin/bash

# Install OpenJDK 17 JRE Headless
sudo apt install openjdk-17-jre-headless -y

# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y
```

![Jenkins Configuration](9-configure%20jenkins.png)

## Installing Docker
Docker is a platform that enables developers to create, deploy, and run applications in containers. Containers are lightweight, portable, and ensure consistency across multiple environments. This project uses Docker to containerize applications and services, making them easier to manage and deploy.

### Docker Installation Script
```bash
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Docker
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo chmod 666 /var/run/docker.sock
```

## Setting Up SonarQube
SonarQube is a tool for continuous inspection of code quality. It performs automatic reviews of code to detect bugs, vulnerabilities, and code smells. In this project, SonarQube is used to ensure that the codebase maintains high quality and adheres to best practices.

### SonarQube Installation Command
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

![SonarQube Installation](10-install%20sonar%20qube.png)
![SonarQube Project Setup](11-sonar%20working.png)

## Local Execution
For local development and testing, the application is deployed on a local machine using tools like Mobaxterm for SSH connections, NVM for managing Node.js versions, and other necessary configurations. This ensures that the application runs correctly in a controlled local environment before being deployed to production.

### Steps for Local Execution
1. **Connect to the Machine**: Use Mobaxterm to connect to your Ubuntu machine using an SSH key.

![Connect to Machine](2-install%20npm.png)

2. **Clone the Repository**:
   ```bash
   git clone https://github.com/jomaa-ahmed/3tierAppDevops.git
   cd 3tierAppDevops
   ```

3. **Install Node.js using NVM**:
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
   nvm install 20
   node -v
   npm -v
   ```

4. **Obtain API keys**: 
   - Create an account on Cloudinary and obtain your cloud name, API key, and secret.
   - Create an account on Mapbox and obtain your public access token.
   - Sign up for MongoDB Atlas and create a database. Retrieve your connection URL.

![Cloudinary Configuration](3-configure%20cloudinary.png)
![Mapbox Account](4-mapbox%20account.png)
![MongoDB Atlas](5-mongodb%20atlas.png)

5. **Create a `.env` file**:
   ```bash
   vi .env
   ```
   Add the following lines to the file and replace placeholders with your actual values:
   ```env
   CLOUDINARY_CLOUD_NAME=[Your Cloudinary Cloud Name]
   CLOUDINARY_KEY=[Your Cloudinary Key]
   CLOUDINARY_SECRET=[Your Cloudinary Secret]
   MAPBOX_TOKEN=[Your Mapbox Token]
   DB_URL=[Your MongoDB Atlas Connection URL]
   SECRET=[Your Chosen Secret Key]
   ```

6. **Install project dependencies**:
   ```bash
   npm install
   ```

7. **Start the application**:
   ```bash
   npm start
   ```

8. **Access the app**: Open a web browser and navigate to `http://VM_IP:3000` (replace VM_IP with the IP address of your Ubuntu machine).

![Access the App](7-npm%20i%20local.png)

## Dev Deployment Pipeline
The development deployment pipeline automates the process of building, testing, and deploying the application in a development environment. It ensures that code changes are integrated continuously, tested for quality, and deployed to a development server for further testing and validation.

### Git Checkout Stage
In this stage, the Jenkins pipeline checks out the code from the Git repository.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jomaa-ahmed/3tierAppDevops.git'
            }
        }
    }
}
```

### Install Dependencies Stage
This stage installs the necessary dependencies for the application using npm.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('install dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```

### Unit Tests Stage
In this stage, the pipeline runs unit tests to ensure the application is working as expected.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Unit tests') {
            steps {
                sh "npm test"
            }
        }
    }
}
```

### Trivy FS Scan Stage
This stage performs a filesystem scan using Trivy to identify vulnerabilities.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Trivy fs scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
    }
}
```

### SonarQube Analysis Stage
In this stage, the pipeline runs a SonarQube analysis to check for code quality and security issues.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar

-scanner'
    }

    stages {
        stage('SonarQube analysis') {
            steps {
                sh "${SCNNAMER_HOME}/bin/sonar-scanner"
            }
        }
    }
}
```

### Build Stage
This stage builds the application to prepare it for deployment.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Build') {
            steps {
                sh "npm run build"
            }
        }
    }
}
```

### Deploy to Dev Stage
In this stage, the application is deployed to the development environment.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Deploy to Dev') {
            steps {
                sh "docker-compose -f docker-compose.dev.yml up -d"
            }
        }
    }
}
```

## Production Deployment Pipeline
The production deployment pipeline automates the process of deploying the application to the production environment. It ensures that the application is built, tested, and deployed in a consistent and reliable manner.

### Build and Push Docker Image Stage
In this stage, the Docker image is built and pushed to a container registry.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t myapp:latest ."
            }
        }
        
        stage('Push Docker Image') {
            steps {
                sh "docker tag myapp:latest myregistry/myapp:latest"
                sh "docker push myregistry/myapp:latest"
            }
        }
    }
}
```

### Deploy to Production Stage
This stage deploys the application to the production environment using the Docker image.

```groovy
pipeline {
    agent any
    
    tools{
        nodejs 'node21'
    }
    environment{
        SCNNAMER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Deploy to Production') {
            steps {
                sh "docker-compose -f docker-compose.prod.yml up -d"
            }
        }
    }
}
```

## Additional Configurations

### Setting Up Cloudinary
1. Create an account on Cloudinary.
2. Obtain your cloud name, API key, and secret.
3. Update your `.env` file with the obtained values.

### Setting Up Mapbox
1. Create an account on Mapbox.
2. Obtain your public access token.
3. Update your `.env` file with the obtained token.

### Setting Up MongoDB Atlas
1. Sign up for MongoDB Atlas.
2. Create a new database.
3. Retrieve your connection URL.
4. Update your `.env` file with the connection URL.

### Updating Jenkins Credentials
1. Go to Jenkins Dashboard.
2. Click on "Manage Jenkins" > "Manage Credentials".
3. Add new credentials for Cloudinary, Mapbox, and MongoDB Atlas.

### Configuring Jenkins for Node.js
1. Go to Jenkins Dashboard.
2. Click on "Manage Jenkins" > "Global Tool Configuration".
3. Add Node.js installation under "NodeJS installations".

## Conclusion
This documentation provides a detailed guide for setting up and deploying a 3-tier full stack project. By following the outlined steps, you can ensure a smooth integration of Jenkins, Docker, SonarQube, Cloudinary, Mapbox, and MongoDB Atlas, resulting in a well-organized and efficient development and deployment process.

Feel free to adjust and expand upon these configurations to fit your specific project requirements.
```

You can copy and paste this into your Markdown file. Let me know if you need any further adjustments!