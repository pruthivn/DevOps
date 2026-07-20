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

## Interview Questions form DigiCert

### 1. what is Static pod?
A. Static Pods are managed directly by the kubelet on a specific node, not by the Kubernetes API server or a Deployment.
we put the pod yaml file in **/etc/kubernetes/manifests** directory kubelet watches this directory and automatically creates or restarts the Pods if they stop.

Because static pods do not depend on the API server to run, they are perfect for running the software needed to start the API server. 

Static Pods are commonly used for control plane components like the API server, etcd, controller manager, and scheduler in self-managed Kubernetes clusters.

### 2. What is PDB(Pod Disruption Budget)?
A. A Pod Disruption Budget (PDB) is a Kubernetes resource that ensures a minimum number of application pods remain available during voluntary disruptions, such as node maintenance, cluster upgrades, or node draining.

in the below yaml minAvailabel is 2 it will maintain 2 pods availabe if we want remove that pods Pod Disruption Budget blocks the eviction until replacement pods(new pods) become available.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-frontend
```
