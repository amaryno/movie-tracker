# Cloud Deployment Guide - Linode Kubernetes Engine (LKE)

## Overview

This guide walks you through deploying the Movie Tracker application to Linode Kubernetes Engine (LKE), a managed Kubernetes service.

## Prerequisites

### 1. Linode Account Setup

- Create account at [linode.com](https://www.linode.com)
- Add payment method
- Generate API token: Account > API Tokens > Create Personal Access Token

### 2. Required Tools

```bash
# Install Linode CLI
pip3 install linode-cli

# Configure CLI
linode-cli configure

# Install kubectl (if not already installed)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Helm (for additional deployments)
curl https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz | tar xz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

## Step 1: Create LKE Cluster

### Option A: Using Linode CLI

```bash
# Create LKE cluster (adjust specs as needed)
linode-cli lke cluster-create \
  --label movie-tracker-cluster \
  --region us-east \
  --k8s_version 1.28 \
  --node_pools.type g6-standard-2 \
  --node_pools.count 2 \
  --node_pools.autoscaler.enabled true \
  --node_pools.autoscaler.min 1 \
  --node_pools.autoscaler.max 5

# Get cluster ID from output, then download kubeconfig
CLUSTER_ID="your-cluster-id"
linode-cli lke kubeconfig-view $CLUSTER_ID --text --no-headers | base64 -d > ~/.kube/config-lke

# Switch to LKE context
export KUBECONFIG=~/.kube/config-lke
kubectl config get-contexts
```

### Option B: Using Linode Cloud Manager (Web UI)

1. Go to Kubernetes section in Linode Cloud Manager
2. Create Cluster:
   - **Cluster Label**: movie-tracker-cluster
   - **Region**: us-east (or closest to you)
   - **Kubernetes Version**: Latest stable (1.28+)
   - **Node Pools**:
     - **Plan**: Linode 4GB (g6-standard-2)
     - **Node Count**: 2
     - **Autoscaler**: Enabled (min: 1, max: 5)
3. Click Create Cluster
4. Download kubeconfig file when ready

## Step 2: Configure kubectl for LKE

```bash
# Download kubeconfig (if using web UI)
# File will be downloaded as: lke-cluster-config.yaml

# Set kubeconfig
export KUBECONFIG=/path/to/lke-cluster-config.yaml

# Verify connection
kubectl cluster-info
kubectl get nodes
```

## Step 3: Deploy Application to LKE

### Deploy Using Our Cloud-Optimized Manifest

```bash
# Apply our cloud deployment
kubectl apply -f k8s/cloud-deployment.yaml

# Check deployment status
kubectl get pods -n movie-tracker
kubectl get services -n movie-tracker

# Get external IP for frontend (this may take a few minutes)
kubectl get service frontend -n movie-tracker -w
```

### Alternative: Deploy with Separate Files

```bash
# Deploy individual components
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/frontend-deployment.yaml
```

## Step 4: Configure Domain and SSL (Optional)

### Option A: Using LoadBalancer IP

```bash
# Get the external IP
EXTERNAL_IP=$(kubectl get service frontend -n movie-tracker -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Access your app at: http://$EXTERNAL_IP"
```

### Option B: Using Custom Domain with cert-manager

```bash
# Install cert-manager for SSL certificates
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create ClusterIssuer for Let's Encrypt
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

# Install Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml

# Create Ingress with SSL
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: movie-tracker-ingress
  namespace: movie-tracker
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/redirect-to-https: "true"
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: movie-tracker-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
EOF
```

## Step 5: Monitoring and Logging Setup

### Install Prometheus and Grafana

```bash
# Add Helm repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin123

# Get Grafana admin password
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward to access Grafana
kubectl port-forward --namespace monitoring svc/prometheus-grafana 3000:80
# Access Grafana at http://localhost:3000 (admin/admin123)
```

## Step 6: Application Scaling and Updates

### Horizontal Pod Autoscaling

```bash
# Create HPA for backend
kubectl autoscale deployment backend --cpu-percent=70 --min=2 --max=10 -n movie-tracker

# Create HPA for frontend
kubectl autoscale deployment frontend --cpu-percent=70 --min=2 --max=10 -n movie-tracker

# Check HPA status
kubectl get hpa -n movie-tracker
```

### Rolling Updates

```bash
# Update backend image
kubectl set image deployment/backend backend=amarynow/movie-tracker-backend:v2.0 -n movie-tracker

# Check rollout status
kubectl rollout status deployment/backend -n movie-tracker

# Rollback if needed
kubectl rollout undo deployment/backend -n movie-tracker
```

## Cost Optimization

### 1. Resource Limits

Add resource limits to deployments to control costs:

```yaml
resources:
 limits:
  cpu: 500m
  memory: 512Mi
 requests:
  cpu: 250m
  memory: 256Mi
```

### 2. Node Pool Management

```bash
# Scale down during off-hours
linode-cli lke pool-update $CLUSTER_ID $POOL_ID --count 1

# Scale up for peak times
linode-cli lke pool-update $CLUSTER_ID $POOL_ID --count 3
```

### 3. Spot Instances (when available)

- Use preemptible nodes for cost savings
- Ensure applications can handle node interruptions

## Troubleshooting

### Common Issues

1. **Pods stuck in Pending**:

   ```bash
   kubectl describe pod <pod-name> -n movie-tracker
   # Check for resource constraints or node capacity
   ```

2. **LoadBalancer stuck in Pending**:

   ```bash
   kubectl describe service frontend -n movie-tracker
   # Linode LB may take 5-10 minutes to provision
   ```

3. **Database connection issues**:
   ```bash
   kubectl logs deployment/backend -n movie-tracker
   # Check PostgreSQL service and environment variables
   ```

### Useful Commands

```bash
# Get all resources in namespace
kubectl get all -n movie-tracker

# Check cluster events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check node resources
kubectl top nodes

# Check pod resources
kubectl top pods -n movie-tracker

# Get detailed cluster info
kubectl cluster-info dump > cluster-info.txt
```

## Security Best Practices

1. **Network Policies**: Implement network segmentation
2. **RBAC**: Use Role-Based Access Control
3. **Pod Security Standards**: Enforce security contexts
4. **Secrets Management**: Use Kubernetes secrets for sensitive data
5. **Image Scanning**: Scan images for vulnerabilities
6. **Regular Updates**: Keep cluster and nodes updated

## Backup Strategy

### Database Backups

```bash
# Create backup job
kubectl create job --from=cronjob/postgres-backup backup-$(date +%Y%m%d) -n movie-tracker

# Restore from backup
kubectl exec -it postgres-pod -n movie-tracker -- pg_restore -U movieuser -d moviesdb /backups/backup.sql
```

## Cleanup

### Delete Everything

```bash
# Delete application
kubectl delete namespace movie-tracker

# Delete cluster (THIS WILL DELETE EVERYTHING!)
linode-cli lke cluster-delete $CLUSTER_ID
```

## Next Steps

1. **Set up CI/CD pipeline** to deploy directly to LKE
2. **Implement blue-green deployments** for zero-downtime updates
3. **Add comprehensive monitoring** with Prometheus alerts
4. **Set up log aggregation** with ELK stack
5. **Implement backup automation** for data persistence
6. **Configure disaster recovery** procedures
