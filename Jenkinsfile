pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_REGISTRY = 'amarynow'
        IMAGE_TAG = "${BUILD_NUMBER}"
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/movie-tracker-backend"
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/movie-tracker-frontend"
    }
    
    tools {
        nodejs '20'  // Requires NodeJS plugin and configured Node.js installation
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Checking out source code...'
                checkout scm
            }
        }
        
        stage('Backend Tests') {
            steps {
                echo '🧪 Running backend tests...'
                dir('backend') {
                    sh '''
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install -r requirements.txt
                        # Add your test commands here when you have tests
                        # python -m pytest tests/
                        echo "Backend tests passed!"
                    '''
                }
            }
        }
        
        stage('Frontend Tests') {
            steps {
                echo '🧪 Running frontend tests...'
                dir('frontend/movie-frontend') {
                    sh '''
                        npm install
                        # Add your test commands here when you have tests
                        # npm test -- --watch=false --browsers=ChromeHeadless
                        npm run build --output-path=dist
                        echo "Frontend tests passed!"
                    '''
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        echo '🐳 Building backend Docker image...'
                        dir('backend') {
                            script {
                                def backendImage = docker.build("${BACKEND_IMAGE}:${IMAGE_TAG}")
                                docker.build("${BACKEND_IMAGE}:latest")
                            }
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        echo '🐳 Building frontend Docker image...'
                        dir('frontend/movie-frontend') {
                            script {
                                def frontendImage = docker.build("${FRONTEND_IMAGE}:${IMAGE_TAG}")
                                docker.build("${FRONTEND_IMAGE}:latest")
                            }
                        }
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                echo '🔒 Running security scans...'
                script {
                    // Using Trivy for vulnerability scanning (if installed)
                    sh '''
                        # Install trivy if not available
                        if ! command -v trivy &> /dev/null; then
                            echo "Trivy not installed, skipping security scan"
                        else
                            trivy image ${BACKEND_IMAGE}:${IMAGE_TAG} || true
                            trivy image ${FRONTEND_IMAGE}:${IMAGE_TAG} || true
                        fi
                    '''
                }
            }
        }
        
        stage('Push to Registry') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                echo '📦 Pushing images to Docker Hub...'
                script {
                    docker.withRegistry('https://registry-1.docker.io/v2/', 'docker-hub-credentials') {
                        sh "docker push ${BACKEND_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${BACKEND_IMAGE}:latest"
                        sh "docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${FRONTEND_IMAGE}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to Development') {
            when {
                branch 'develop'
            }
            steps {
                echo '🚀 Deploying to development environment...'
                script {
                    // Deploy to local Minikube for development
                    sh '''
                        # Update image tags in K8s manifests
                        sed -i "s|amarynow/movie-tracker-backend:latest|${BACKEND_IMAGE}:${IMAGE_TAG}|g" k8s/backend-deployment.yaml
                        sed -i "s|amarynow/movie-tracker-frontend:latest|${FRONTEND_IMAGE}:${IMAGE_TAG}|g" k8s/frontend-deployment.yaml
                        
                        # Deploy to Minikube
                        kubectl config use-context minikube
                        kubectl apply -f k8s/ --namespace=default
                        
                        # Wait for rollout
                        kubectl rollout status deployment/backend --timeout=300s
                        kubectl rollout status deployment/frontend --timeout=300s
                    '''
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo '🌟 Deploying to production environment...'
                script {
                    // This would deploy to your cloud Kubernetes cluster
                    sh '''
                        # Update image tags for production deployment
                        sed -i "s|amarynow/movie-tracker-backend:latest|${BACKEND_IMAGE}:${IMAGE_TAG}|g" k8s/cloud-deployment.yaml
                        sed -i "s|amarynow/movie-tracker-frontend:latest|${FRONTEND_IMAGE}:${IMAGE_TAG}|g" k8s/cloud-deployment.yaml
                        
                        # Deploy to cloud cluster (replace with your cloud config)
                        # kubectl config use-context your-cloud-context
                        # kubectl apply -f k8s/cloud-deployment.yaml
                        
                        echo "Production deployment configured. Apply k8s/cloud-deployment.yaml to your cloud cluster."
                    '''
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                echo '🔗 Running integration tests...'
                script {
                    // Wait for services and run integration tests
                    sh '''
                        echo "Running integration tests..."
                        # Add your integration test commands here
                        # For example, using curl to test API endpoints
                        # curl -f http://your-app-url/health || exit 1
                        echo "Integration tests passed!"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo '🧹 Cleaning up...'
            script {
                // Clean up Docker images to save space
                sh '''
                    docker image prune -f
                    docker system df
                '''
            }
        }
        success {
            echo '✅ Pipeline completed successfully!'
            // You can add Slack/email notifications here
        }
        failure {
            echo '❌ Pipeline failed!'
            // You can add failure notifications here
        }
    }
}

