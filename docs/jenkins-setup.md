# Jenkins CI/CD Setup Guide for Movie Tracker

## Prerequisites

1. **Jenkins Installation with Plugins**:

   - Docker Pipeline plugin
   - Kubernetes CLI plugin
   - NodeJS plugin
   - Git plugin
   - Pipeline plugin

2. **Required Tools on Jenkins Agent**:
   - Docker
   - kubectl
   - Node.js 20+
   - Python 3.11+

## Jenkins Configuration Steps

### 1. Install Required Plugins

Go to `Manage Jenkins` > `Manage Plugins` > `Available` and install:

- Docker Pipeline
- NodeJS
- Kubernetes CLI
- Pipeline Stage View
- Blue Ocean (optional, for better UI)

### 2. Configure Node.js

1. Go to `Manage Jenkins` > `Global Tool Configuration`
2. Under `NodeJS`, click `Add NodeJS`
3. Name: `20`
4. Version: `NodeJS 20.x.x`
5. Save

### 3. Configure Docker Hub Credentials

1. Go to `Manage Jenkins` > `Manage Credentials`
2. Click `(global)` > `Add Credentials`
3. Kind: `Username with password`
4. Username: `amarynow`
5. Password: `your-docker-hub-password`
6. ID: `docker-hub-credentials`
7. Description: `Docker Hub Registry Credentials`

### 4. Create Pipeline Job

1. New Item > Pipeline
2. Name: `movie-tracker-pipeline`
3. In Pipeline section:
   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: `https://github.com/amaryno/movie-tracker.git`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`

### 5. Configure Kubernetes (if using cloud cluster)

1. Install kubectl on Jenkins agent
2. Add kubeconfig as secret file credential:
   - Go to `Manage Credentials`
   - Add Credentials > Secret file
   - File: Upload your kubeconfig file
   - ID: `kubeconfig`

## Pipeline Features

### Stages Overview

1. **Checkout**: Gets source code from GitHub
2. **Backend Tests**: Runs Python tests (add pytest when ready)
3. **Frontend Tests**: Runs Angular tests (add when ready)
4. **Build Docker Images**: Builds both images in parallel
5. **Security Scan**: Scans images for vulnerabilities with Trivy
6. **Push to Registry**: Pushes to Docker Hub (only on main/develop branches)
7. **Deploy to Development**: Deploys to Minikube (develop branch)
8. **Deploy to Production**: Deploys to cloud cluster (main branch)
9. **Integration Tests**: Runs end-to-end tests

### Branch Strategy

- **develop**: Deploys to local Minikube for testing
- **main**: Deploys to production cloud cluster
- **feature/\***: Only builds and tests, no deployment

### Environment Variables

The pipeline uses these environment variables:

- `DOCKER_REGISTRY`: amarynow
- `IMAGE_TAG`: Build number for versioning
- `BACKEND_IMAGE`: Full backend image name
- `FRONTEND_IMAGE`: Full frontend image name

## Running the Pipeline

### Local Development Setup

1. **Start Jenkins**:

   ```bash
   docker run -d \
     --name jenkins \
     -p 8080:8080 \
     -p 50000:50000 \
     -v jenkins_home:/var/jenkins_home \
     -v /var/run/docker.sock:/var/run/docker.sock \
     jenkins/jenkins:lts
   ```

2. **Get Initial Admin Password**:

   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

3. **Access Jenkins**: http://localhost:8080

### Manual Pipeline Trigger

1. Go to your pipeline job
2. Click `Build Now`
3. Monitor progress in `Blue Ocean` or console output

### Webhook Setup (Optional)

Configure GitHub webhook to trigger builds automatically:

1. Go to GitHub repo > Settings > Webhooks
2. Add webhook: `http://your-jenkins-url/github-webhook/`
3. Content type: `application/json`
4. Events: `Push` and `Pull requests`

## Troubleshooting

### Common Issues

1. **Docker permission denied**:

   ```bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```

2. **Kubectl not found**:

   ```bash
   # Install kubectl on Jenkins agent
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```

3. **Node.js not found**:
   - Check Global Tool Configuration
   - Ensure NodeJS plugin is installed
   - Restart Jenkins after configuration

### Logs and Debugging

- Check console output for detailed error messages
- Use `Blue Ocean` plugin for better visualization
- Enable debug logging in Jenkins system log

## Security Best Practices

1. **Never store secrets in Jenkinsfile**
2. **Use Jenkins credentials store**
3. **Enable CSRF protection**
4. **Use role-based access control**
5. **Keep Jenkins and plugins updated**
6. **Scan Docker images for vulnerabilities**

## Next Steps

1. Add comprehensive test suites
2. Implement infrastructure as code (Terraform)
3. Add monitoring and alerting
4. Set up multi-environment deployments
5. Implement canary deployments
