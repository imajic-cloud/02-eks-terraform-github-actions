# AWS DevOps EKS Project

A production-ready DevOps demonstration project showcasing automated infrastructure provisioning, containerized application deployment, and CI/CD pipeline implementation using AWS cloud services and Kubernetes.

## Overview

This project implements a complete DevOps workflow that automates the entire application lifecycle from code commit to production deployment on AWS Elastic Kubernetes Service (EKS).

## Architecture

```
Developer → GitHub → GitHub Actions → Docker → AWS ECR → Helm → AWS EKS → LoadBalancer → End Users
```

**Workflow Breakdown:**
1. Developer commits code to GitHub repository
2. GitHub Actions CI/CD pipeline is triggered automatically
3. Docker image is built from the application code
4. Image is pushed to AWS Elastic Container Registry (ECR)
5. Helm chart deploys the containerized application
6. Application runs on AWS EKS cluster
7. LoadBalancer exposes the application to users

## Technologies Stack

| Category | Technology |
|----------|-----------|
| **Cloud Provider** | AWS (EKS, ECR, VPC, Load Balancer) |
| **Infrastructure as Code** | Terraform |
| **Container Runtime** | Docker |
| **Container Registry** | AWS ECR |
| **Orchestration** | Kubernetes (AWS EKS) |
| **Package Manager** | Helm 3 |
| **CI/CD** | GitHub Actions |
| **Operating System** | Linux / WSL |
| **Application** | Nginx web server |

## Project Structure

```
devops-eks-project/
│
├── infrastructure/          # Terraform configuration files
│   ├── main.tf             # EKS cluster and VPC configuration
│   ├── variables.tf        # Input variables
│   ├── outputs.tf          # Output values
│   └── terraform.tfvars    # Variable values
│
├── app/                    # Application source code
│   ├── Dockerfile          # Container image definition
│   ├── index.html          # Web application files
│   └── nginx.conf          # Nginx configuration
│
├── helm/                   # Helm chart for Kubernetes deployment
│   ├── Chart.yaml          # Chart metadata
│   ├── values.yaml         # Default configuration values
│   └── templates/          # Kubernetes manifest templates
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
│
└── .github/
    └── workflows/          # CI/CD pipeline definitions
        └── deploy.yml      # GitHub Actions workflow
```

## Prerequisites

Before running this project, ensure you have the following installed and configured:

- **AWS CLI** (v2.x or higher) - configured with appropriate credentials
- **Terraform** (v1.0 or higher)
- **Docker** (v20.x or higher)
- **kubectl** (compatible with your EKS version)
- **Helm** (v3.x)
- **AWS Account** with permissions for EKS, ECR, VPC, and IAM

## Getting Started

### 1. Infrastructure Provisioning

Navigate to the infrastructure directory and initialize Terraform:

```bash
cd infrastructure
terraform init
terraform plan
terraform apply
```

This will create:
- AWS VPC with public and private subnets
- EKS cluster with managed node groups
- IAM roles and security groups
- ECR repository for Docker images

### 2. Configure kubectl

After EKS cluster creation, configure kubectl to interact with your cluster:

```bash
aws eks update-kubeconfig --name <cluster-name> --region <aws-region>
```

Verify the connection:

```bash
kubectl get nodes
```

### 3. Build and Push Docker Image

Build the Docker image locally:

```bash
cd app
docker build -t devops-eks-app .
```

Authenticate Docker to AWS ECR:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

Tag and push the image:

```bash
docker tag devops-eks-app:latest <account-id>.dkr.ecr.<region>.amazonaws.com/devops-eks-app:latest
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/devops-eks-app:latest
```

### 4. Deploy with Helm

Deploy the application to your EKS cluster:

```bash
cd helm
helm install project1 ./devops-eks-app
```

Check deployment status:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

### 5. Access the Application

Get the LoadBalancer URL:

```bash
kubectl get service project1 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Open the URL in your browser to access the deployed application.

## CI/CD Pipeline

The GitHub Actions workflow automates the entire deployment process:

### Pipeline Stages

1. **Code Checkout** - Pulls the latest code from the repository
2. **Docker Build** - Creates a container image from the Dockerfile
3. **ECR Push** - Uploads the image to AWS ECR with version tagging
4. **Helm Deploy** - Updates the Kubernetes deployment with the new image
5. **Verification** - Confirms successful deployment

### Triggering the Pipeline

The pipeline automatically runs on:
- Push to `main` branch
- Pull request merge
- Manual workflow dispatch

## Kubernetes Operations

### Scaling the Application

Scale the deployment to handle increased traffic:

```bash
# Scale to 3 replicas
kubectl scale deployment project1 --replicas=3

# Verify scaling
kubectl get pods
```

### Rolling Updates

Update the application with zero downtime:

```bash
helm upgrade project1 ./devops-eks-app --set image.tag=v2.0
```

### View Logs

Check application logs:

```bash
kubectl logs -f deployment/project1
```

### Monitor Resources

```bash
kubectl top nodes
kubectl top pods
```

## Kubernetes Features Implemented

- ✅ **Helm Chart Deployment** - Package manager for Kubernetes
- ✅ **Rolling Updates** - Zero-downtime deployments
- ✅ **LoadBalancer Service** - External traffic routing
- ✅ **Horizontal Pod Autoscaling** - Automatic scaling based on metrics
- ✅ **Resource Limits** - CPU and memory constraints
- ✅ **Health Checks** - Liveness and readiness probes
- ✅ **ConfigMaps** - External configuration management
- ✅ **Secrets** - Secure credential storage

## Cleanup

### Uninstall Helm Release

Before destroying infrastructure, remove the Helm deployment:

```bash
helm uninstall project1
```

Verify all resources are deleted:

```bash
kubectl get all
```

### Destroy Infrastructure

Remove all AWS resources created by Terraform:

```bash
cd infrastructure
terraform destroy
```

**Warning:** This will permanently delete your EKS cluster, VPC, and all associated resources.

## Security Best Practices

This project implements several security measures:

- IAM roles with least privilege principle
- Private subnets for worker nodes
- Security groups with restricted ingress/egress
- ECR image scanning for vulnerabilities
- Kubernetes RBAC for access control
- Secrets stored in AWS Secrets Manager (production)

## Cost Optimization

To minimize AWS costs:

- Use Spot Instances for non-production workloads
- Enable cluster autoscaling to scale down during low usage
- Set resource requests and limits appropriately
- Delete unused ECR images regularly
- Use smaller instance types for development

## Troubleshooting

### Common Issues

**Issue:** kubectl cannot connect to cluster
```bash
# Solution: Update kubeconfig
aws eks update-kubeconfig --name <cluster-name> --region <region>
```

**Issue:** Pods stuck in Pending state
```bash
# Check events
kubectl describe pod <pod-name>

# Check node capacity
kubectl describe nodes
```

**Issue:** LoadBalancer not getting external IP
```bash
# Check service events
kubectl describe service project1

# Verify AWS Load Balancer Controller is installed
kubectl get pods -n kube-system
```

## Future Enhancements

- [ ] Implement GitOps with ArgoCD or Flux
- [ ] Add monitoring with Prometheus and Grafana
- [ ] Implement centralized logging with ELK stack
- [ ] Add service mesh (Istio) for advanced traffic management
- [ ] Implement blue-green deployment strategy
- [ ] Add automated security scanning in CI/CD pipeline
- [ ] Implement disaster recovery and backup strategies
- [ ] Add multi-region deployment

## Learning Outcomes

This project demonstrates proficiency in:

- Cloud infrastructure automation with Terraform
- Container orchestration using Kubernetes
- CI/CD pipeline design and implementation
- AWS cloud services (EKS, ECR, VPC, IAM)
- DevOps best practices and tooling
- Infrastructure as Code principles
- Microservices deployment patterns

## Resources

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Author

**DevOps Portfolio Project**

This project demonstrates hands-on experience with modern DevOps practices, cloud-native technologies, and automated infrastructure management. Created as part of a comprehensive DevOps learning journey.

---

## License

This project is open source and available for educational purposes.

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

---

**⭐ If you find this project helpful, please consider giving it a star!**
