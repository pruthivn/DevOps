# Kubernetes

This directory contains Kubernetes-related DevOps interview questions and resources.

## Kubernetes / EKS Q&A from ADP

## 18. ECR vs EKS
- ECR stores container images.
- EKS runs Kubernetes workloads.

## 26. Monolith to EKS Migration
1. Assess application
2. Containerize
3. Push image to ECR
4. Create EKS cluster
5. Configure networking
6. Deploy application
7. Configure monitoring

## 27. Kubernetes Cluster Upgrade
- Upgrade control plane
- Upgrade node groups
- Validate workloads

## 28. EKS Downgrade
Not supported. Create a new cluster and migrate workloads.

## 29. Production Scenarios

### CPU Spike
- Check metrics
- Analyze logs
- Scale resources
- Rollback if required

### Application Inaccessible
Check:
DNS → Load Balancer → Ingress → Service → Pods → Database
