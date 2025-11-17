# 🎬 Movie Tracker - DevOps Bootcamp

A comprehensive full-stack demo application designed for learning modern DevOps practices, including containerization, orchestration, CI/CD, and cloud deployment.

---

## 🚀 Tech Stack

- **Frontend**: Angular 17+ with TypeScript  
- **Backend**: FastAPI (Python) with SQLAlchemy
- **Database**: PostgreSQL 13
- **Containerization**: Docker & Docker Compose
- **Orchestration**: Kubernetes (Minikube + Cloud)
- **CI/CD**: Jenkins Pipeline
- **Cloud**: Linode Kubernetes Engine (LKE)
- **Registry**: Docker Hub
- **Monitoring**: Prometheus & Grafana (optional)

---

## 📚 What You'll Learn

This bootcamp covers:
- 🐳 **Docker**: Multi-stage builds, optimization, networking
- 📦 **Docker Compose**: Local development environment  
- ☸️ **Kubernetes**: Deployments, services, ingress, scaling
- 🔄 **CI/CD**: Jenkins pipeline with automated testing and deployment
- ☁️ **Cloud Deployment**: Production-ready Kubernetes on Linode
- 📊 **Monitoring**: Application observability and alerting
- 🔒 **Security**: Container scanning, secrets management
- 📈 **Scaling**: Horizontal pod autoscaling, load balancing

---

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Angular       │────│   FastAPI       │────│   PostgreSQL    │
│   Frontend      │    │   Backend       │    │   Database      │  
│   (Port 80)     │    │   (Port 8000)   │    │   (Port 5432)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Kubernetes    │
                    │    Cluster      │ 
                    └─────────────────┘
```

---

## 🗂️ Project Structure

```
movie-tracker/
├── backend/                    # FastAPI backend application
│   ├── main.py                # Main application file
│   ├── requirements.txt       # Python dependencies
│   └── dockerfile            # Backend container build
├── frontend/                  # Angular frontend application
│   └── movie-frontend/       
│       ├── src/              # Angular source code
│       ├── package.json      # Node.js dependencies
│       ├── dockerfile        # Frontend container build
│       └── docker-entrypoint.sh # Runtime configuration
├── k8s/                       # Kubernetes manifests
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── postgres-deployment.yaml  
│   ├── postgres-service.yaml
│   └── cloud-deployment.yaml    # All-in-one cloud deployment
├── docs/                      # Documentation
│   ├── jenkins-setup.md       # Jenkins CI/CD guide
│   ├── linode-deployment.md   # Cloud deployment guide
│   └── deployment-checklist.md # Go-live checklist
├── docker-compose.yml         # Local development environment
├── docker-compose.jenkins.yml # Jenkins setup
└── Jenkinsfile               # CI/CD pipeline definition
```

---

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- kubectl
- Minikube (for local K8s)
- Node.js 20+ & Python 3.11+

### Local Development
```bash
# Clone the repository
git clone https://github.com/amaryno/movie-tracker.git
cd movie-tracker

# Start all services
docker compose up --build

# Access the application
# Frontend: http://localhost:4200
# Backend API: http://localhost:8000
# Swagger UI: http://localhost:8000/docs
```

### Kubernetes (Local)
```bash
# Start Minikube
minikube start

# Configure Docker environment
eval $(minikube docker-env)

# Build images
docker build -t movie-backend:latest ./backend
docker build -t movie-frontend:latest ./frontend/movie-frontend

# Deploy to Kubernetes
kubectl apply -f k8s/

# Access application
minikube service frontend
```

### Production Deployment
See [Linode Deployment Guide](./docs/linode-deployment.md) for complete cloud deployment instructions.

---

## 🔄 CI/CD Pipeline

Our Jenkins pipeline includes:

1. **Code Checkout** - Get latest source code
2. **Testing** - Run backend and frontend tests
3. **Docker Build** - Create optimized container images
4. **Security Scan** - Vulnerability scanning with Trivy
5. **Registry Push** - Push images to Docker Hub
6. **Deploy Dev** - Auto-deploy to development cluster
7. **Deploy Prod** - Manual promotion to production
8. **Integration Tests** - End-to-end testing

### Pipeline Stages
```groovy
pipeline {
  stages {
    stage('Checkout') { ... }
    stage('Backend Tests') { ... }
    stage('Frontend Tests') { ... }
    stage('Build Docker Images') { ... }
    stage('Security Scan') { ... }
    stage('Push to Registry') { ... }
    stage('Deploy to Development') { ... }
    stage('Deploy to Production') { ... }
    stage('Integration Tests') { ... }
  }
}
```

---

## 📊 Available Images

Public Docker images are available on Docker Hub:
- **Backend**: `amarynow/movie-tracker-backend:latest`
- **Frontend**: `amarynow/movie-tracker-frontend:latest`

---

## 🛠️ Available Scripts

### Development Commands
```bash
# Local development with hot reload
npm run dev                    # Start frontend dev server
python backend/main.py         # Start backend dev server

# Docker commands  
docker compose up --build      # Build and start all services
docker compose down            # Stop all services
docker compose logs -f         # View logs

# Kubernetes commands
kubectl get pods               # Check pod status
kubectl logs deployment/backend # View backend logs
minikube service frontend      # Access frontend
```

### Build Commands
```bash
# Build production images
docker build -t amarynow/movie-tracker-backend:latest ./backend
docker build -t amarynow/movie-tracker-frontend:latest ./frontend/movie-frontend

# Push to registry
docker push amarynow/movie-tracker-backend:latest
docker push amarynow/movie-tracker-frontend:latest
```

### Deployment Commands
```bash
# Deploy to Minikube
kubectl apply -f k8s/

# Deploy to cloud cluster
kubectl apply -f k8s/cloud-deployment.yaml

# Check deployment status
kubectl rollout status deployment/backend -n movie-tracker
kubectl rollout status deployment/frontend -n movie-tracker
```

---

## 📖 Documentation

- **[Jenkins Setup Guide](./docs/jenkins-setup.md)** - Complete CI/CD pipeline setup
- **[Linode Deployment Guide](./docs/linode-deployment.md)** - Cloud deployment instructions  
- **[Deployment Checklist](./docs/deployment-checklist.md)** - Production readiness checklist

---

## 🧪 Testing

### Backend Tests
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m pytest tests/  # When tests are added
```

### Frontend Tests  
```bash
cd frontend/movie-frontend
npm install
npm test                 # Unit tests
npm run e2e             # End-to-end tests (when configured)
```

### Integration Tests
```bash
# Test API endpoints
curl -f http://localhost:8000/movies
curl -X POST http://localhost:8000/movies -H "Content-Type: application/json" -d '{"title":"Test Movie"}'
```

---

## 🔧 Configuration

### Environment Variables

**Backend** (`backend/.env`):
```bash
DB_HOST=postgres
DB_PORT=5432  
DB_NAME=moviesdb
DB_USER=movieuser
DB_PASSWORD=password123
```

**Frontend** (`frontend/movie-frontend/src/assets/env.js`):
```javascript
window.ENV = {
  API_URL: 'http://localhost:8000'
};
```

### Kubernetes Configuration

Key configuration files:
- `k8s/cloud-deployment.yaml` - Complete cloud deployment
- `k8s/backend-deployment.yaml` - Backend service configuration
- `k8s/frontend-deployment.yaml` - Frontend service configuration

---

## 🔒 Security

### Implemented Security Measures
- **Container Scanning**: Trivy vulnerability scanning in CI pipeline
- **Non-root Containers**: All containers run as non-root users  
- **Resource Limits**: CPU and memory limits defined
- **Network Policies**: Kubernetes network segmentation (optional)
- **Secrets Management**: Kubernetes secrets for sensitive data

### Security Checklist
- [ ] Update base images regularly
- [ ] Scan for vulnerabilities
- [ ] Use Kubernetes secrets
- [ ] Implement RBAC
- [ ] Enable network policies
- [ ] Configure pod security standards

---

## 📈 Monitoring

### Metrics & Observability
- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboards  
- **Health Checks**: Kubernetes liveness and readiness probes
- **Logging**: Structured JSON logs
- **Alerting**: Custom alert rules for critical metrics

### Available Dashboards
- Application performance metrics
- Infrastructure resource usage
- Database performance  
- API response times and error rates

---

## 🚀 Scaling

### Horizontal Pod Autoscaler (HPA)
```bash
# Auto-scale based on CPU usage
kubectl autoscale deployment backend --cpu-percent=70 --min=2 --max=10 -n movie-tracker
kubectl autoscale deployment frontend --cpu-percent=70 --min=2 --max=10 -n movie-tracker
```

### Manual Scaling
```bash  
# Scale backend pods
kubectl scale deployment backend --replicas=5 -n movie-tracker

# Scale frontend pods  
kubectl scale deployment frontend --replicas=3 -n movie-tracker
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🎯 Bootcamp Progress

### ✅ Completed Modules
- [x] **Module 1**: Docker Containerization
- [x] **Module 2**: Docker Compose Local Development
- [x] **Module 3**: Kubernetes Orchestration (Minikube)
- [x] **Module 4**: Container Registry (Docker Hub)
- [x] **Module 5**: CI/CD Pipeline (Jenkins)
- [x] **Module 6**: Cloud Deployment (Linode LKE)
- [x] **Module 7**: Production Readiness

### 🎓 Skills Gained
- Container orchestration with Kubernetes
- CI/CD pipeline development
- Cloud deployment and management
- Infrastructure as Code
- Monitoring and observability
- Security best practices
- DevOps toolchain integration

---

## 🆘 Troubleshooting

### Common Issues

**Docker Build Fails**:
```bash
# Clear Docker cache
docker system prune -a
```

**Kubernetes Pod CrashLoopBackOff**:
```bash  
# Check pod logs
kubectl logs <pod-name> -n movie-tracker
kubectl describe pod <pod-name> -n movie-tracker
```

**LoadBalancer Pending**:
```bash
# Check cloud provider limits and quotas
kubectl describe service frontend -n movie-tracker
```

**Database Connection Issues**:
```bash
# Verify service names and environment variables
kubectl get services -n movie-tracker
kubectl exec -it <backend-pod> -n movie-tracker -- env | grep DB_
```

For more detailed troubleshooting, see the [Deployment Checklist](./docs/deployment-checklist.md).

---

## 🌟 Next Steps

After completing this bootcamp, consider exploring:
- **Service Mesh**: Istio for advanced traffic management
- **GitOps**: ArgoCD or Flux for declarative deployments
- **Advanced Monitoring**: Jaeger for distributed tracing  
- **Infrastructure as Code**: Terraform for cloud resource management
- **Security**: Falco for runtime security monitoring
- **Cost Optimization**: Kubernetes resource optimization strategies

---

**🎉 Congratulations on completing the Movie Tracker DevOps Bootcamp!**

You now have hands-on experience with the complete DevOps lifecycle from development to production deployment. These skills are directly applicable to real-world enterprise environments.