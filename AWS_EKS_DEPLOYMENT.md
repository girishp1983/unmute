# AWS EKS Deployment Guide for Unmute with Groq API

## Prerequisites

### 1. AWS Requirements
- AWS CLI configured with appropriate permissions
- EKS cluster with G6e.12xlarge nodes in us-east-1
- ALB Ingress Controller installed
- NVIDIA device plugin for GPU support
- kubectl configured for your EKS cluster

### 2. Required Resources
- **Groq API Key** - Sign up at https://console.groq.com/
- **Domain name** (optional) - For custom ingress
- **SSL Certificate** - AWS ACM certificate for HTTPS

### 3. Docker Images
You'll need to build and push the following Docker images:
- `unmute-backend:latest`
- `unmute-frontend:latest`
- Kyutai STT/TTS images (from Kyutai team)

## EKS Cluster Setup

### 1. Create EKS Cluster with G6e.12xlarge Nodes

```bash
# Create cluster
eksctl create cluster \
  --name unmute-cluster \
  --region us-east-1 \
  --nodegroup-name gpu-nodes \
  --node-type g6e.12xlarge \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Install NVIDIA device plugin
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.0/nvidia-device-plugin.yml

# Install ALB Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.4/docs/install/iam_policy.json
```

### 2. Build and Push Docker Images

```bash
# Build backend image
docker build -f Dockerfile.backend -t unmute-backend:latest .

# Build frontend image  
docker build -f Dockerfile.frontend -t unmute-frontend:latest .

# Tag and push to ECR (replace with your registry)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com

docker tag unmute-backend:latest YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/unmute-backend:latest
docker tag unmute-frontend:latest YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/unmute-frontend:latest

docker push YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/unmute-backend:latest
docker push YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/unmute-frontend:latest
```

## Deployment Steps

### 1. Update Configuration Files

Edit the following files with your specific values:

**k8s/configmap.yaml:**
```yaml
stringData:
  GROQ_API_KEY: "YOUR_ACTUAL_GROQ_API_KEY"
```

**k8s/ingress.yaml:**
```yaml
annotations:
  alb.ingress.kubernetes.io/certificate-arn: "arn:aws:acm:us-east-1:YOUR_ACCOUNT:certificate/YOUR_CERT_ID"
spec:
  rules:
  - host: unmute.yourdomain.com
```

**Backend and Frontend Deployment Images:**
Update image references in deployment files to point to your ECR repositories.

### 2. Deploy to Kubernetes

```bash
# Apply all manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/redis-deployment.yaml
kubectl apply -f k8s/stt-deployment.yaml
kubectl apply -f k8s/tts-deployment.yaml
kubectl apply -f k8s/voice-cloning-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/ingress.yaml
```

### 3. Verify Deployment

```bash
# Check all pods are running
kubectl get pods -n unmute

# Check services
kubectl get svc -n unmute

# Check ingress
kubectl get ingress -n unmute

# Get ALB endpoint
kubectl get ingress unmute-ingress -n unmute -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## Required Information for Deployment

To deploy this application, you need to provide:

1. **Groq API Key** - Get from https://console.groq.com/
2. **AWS Account ID** - Your AWS account number
3. **Domain name** (optional) - For custom URL
4. **SSL Certificate ARN** - From AWS Certificate Manager
5. **ECR Repository URLs** - For storing Docker images

## Estimated Costs

**Monthly AWS costs (us-east-1):**
- 2x G6e.12xlarge instances: ~$3,000-4,000/month
- ALB: ~$20/month
- EKS control plane: ~$75/month
- Storage and networking: ~$100/month

**Total: ~$3,200-4,200/month**

## Resource Requirements

**Per G6e.12xlarge instance:**
- 48 vCPUs
- 192 GB RAM
- 4x NVIDIA L40S GPUs
- 3.8 TB NVMe SSD

**Service allocation:**
- STT: 2 GPUs, 8GB RAM
- TTS: 2 GPUs, 8GB RAM  
- Voice Cloning: 2 GPUs, 8GB RAM
- Backend: 2GB RAM, 1 CPU
- Frontend: 1GB RAM, 0.5 CPU

## Scaling Configuration

The deployment includes:
- Backend: 2 replicas (can auto-scale)
- Frontend: 2 replicas (can auto-scale)
- GPU services: 1 replica each (manual scaling)

## Monitoring and Logging

- CloudWatch integration available
- Prometheus metrics exposed
- Grafana dashboards included
- ALB access logs to S3

## Security Considerations

- API keys stored in Kubernetes secrets
- Network policies for pod-to-pod communication
- ALB with SSL termination
- Private subnets for GPU nodes
- IAM roles for service accounts

## Next Steps

1. Set up your AWS environment with the prerequisites
2. Build and push Docker images to ECR
3. Update configuration files with your values
4. Deploy using the provided manifests
5. Test the application endpoint