# Peerless DevOps Challenge
### Overview
This document provides comprehensive documentation for deploying the Cats Sinatra application (versions 1.0.0 and 2.0.1) using a zero-downtime deployment strategy on AWS ECS with Docker containers. The deployment is automated through GitHub Actions CI/CD pipeline.

### Architecture Overview
- The solution uses the following components:
- AWS ECS (Fargate): Container orchestration service for fault-tolerant deployment
- Docker: Containerization of the Sinatra application
- GitHub Actions: CI/CD pipeline for automated deployments
- Amazon ECR: Container registry for Docker images
- AWS IAM: Secure role-based access control
- AWS CloudWatch: Monitoring and logging
<img width="970" alt="image" src="https://github.com/user-attachments/assets/2bccccbd-0ca9-43c2-a7bb-9e7d135bb98e" />

### Prerequisites
- Before using this deployment system, ensure you have:
- AWS account with appropriate permissions
- ECS cluster configured ("Staging-Cluster" in our case)
- IAM role with ECS/ECR permissions configured in GitHub Secrets as ROLE_TO_ASSUME
- Docker installed locally for development/testing
- GitHub repository with the Cats application code

### Deployment Workflow
Initial Setup (Version 1.0.0)
- Prepare the ECS Environment:
- Create an ECS cluster named "Staging-Cluster"
- Configure ECS service with application load balancer
- Set up task definitions with health checks

### First Deployment:
- Push code to the v1.0.0 branch
- GitHub Actions workflow will automatically:
  - Build Docker image
  - Push to Amazon ECR
    <img width="998" alt="image" src="https://github.com/user-attachments/assets/10fbfa0d-4100-4c68-bb54-8ee913b44e26" />

  - Update ECS task definition
  - Deploy to ECS with zero downtime

### Upgrading to Version 2.0.1
- Merge changes to the main branch
- Tag the release as v2.0.1
- The same GitHub Actions workflow will:
  - Build new Docker image
  - Perform blue-green deployment
  - Verify health checks
  - Route traffic to new version
  - Drain connections from old version

### Zero-Downtime Deployment Strategy
The solution implements zero-downtime deployment through:
- ECS Rolling Updates: ECS automatically replaces tasks in batches
- Health Checks: Ensures new containers are healthy before receiving traffic
- Connection Draining: Allows in-flight requests to complete
- Load Balancer Integration: Smoothly transitions traffic between old and new versions

Key parameters configured:
- wait-for-service-stability: true in GitHub Actions
- Minimum healthy percent = 100%
- Maximum percent = 200%
- Health check grace period = 60 seconds

### Monitoring and Observability
CloudWatch Metrics
- CPU/Memory utilization
- Request counts
- Error rates
- Latency metrics

### Logging
Container logs streamed to CloudWatch Logs
- Access logs from load balancer
- Custom Metrics

### Rollback Procedure
To rollback to previous version:
- Manual Rollback:
  - Re-run previous successful GitHub Actions workflow
  - Use ECS console to revert to previous task definition
- Automated Rollback (if health checks fail):
  - ECS will automatically stop deployment if health checks fail
  - CloudWatch alarms can trigger rollback automation

### Tools and Technologies
- AWS ECS
  - Purpose: Container orchestration
  - Why Chosen: Fully managed, integrates with other AWS services
  - Configuration: Fargate launch type for serverless operation
GitHub Actions
  - Purpose: CI/CD pipeline
  - Why Chosen: Tight GitHub integration, easy configuration
  - Workflow: Defined in .github/workflows/deploy.yml
- Docker
  - Purpose: Containerization
  - Why Chosen: Standardization, portability
  - Image: Built from provided Sinatra application

### Pros and Cons of Chosen Tools
- AWS ECS
  - Pros:
    - Fully managed service
    - Tight AWS integration
    - Supports zero-downtime deployments
    - Cost-effective with Fargate
  - Cons:
    - AWS lock-in
    - Less flexible than Kubernetes for complex scenarios
- GitHub Actions
  - Pros:
    - Native GitHub integration
    - Easy YAML configuration
    - Large marketplace of actions
  - Cons:
    - Limited compared to dedicated CI/CD tools
    - Can become complex for large workflows
- Docker
  - Pros:
    - Standardization
    - Isolation of dependencies
    - Reproducible environments
  - Cons:
    - Additional complexity
    - Requires Docker expertise
### Maintenance and Operations
- Routine Tasks
  - Monitor CloudWatch dashboards
  - Review deployment logs
  - Update dependencies in Dockerfile
- Scaling
  - Configure ECS auto-scaling based on CPU/memory metrics
  - Adjust task count based on load balancer metrics
- Cost Optimization
  - Right-size Fargate task CPU/memory
  - Clean up old ECR images
  - Monitor and adjust auto-scaling parameters
