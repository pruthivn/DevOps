## Rapyder company

### 1. i have different slave nodes i want to run different jobs/stages in different nodes?
we can achieve this using node labels in the pipeline.
1. Assign labels to your nodes(Manage Jenkins -> Nodes -> Configure) find the label option add label name e.g., linux docker build-node.

2. use the label in your pipeline 
```groovy
pipeline {
    agent none // No default agent for the whole pipeline
    stages {
        stage('Linux Build') {
            agent { label 'linux docker build-node' }
            steps {
                 sh 'docker build -t my-app .'
            }
        }
        stage('Windows Test') {
            agent { label 'windows-slave' }
            steps {
                bat 'hostname'
            }
        }
    }
}
```

## Delliote (Linkedin)
### 2. Your Jenkins pipeline succeeds, but the latest application version is not reflected in production. How would you troubleshoot? 
1. **Review Jenkins Logs:** Look for errors or warnings in the build and deployment logs.
2. **Check Image Tagging & Caching:** If deploying with :latest, Kubernetes or Docker on the host may not pull the new image if imagePullPolicy is set to IfNotPresent. Always tag builds with immutable Git commit SHAs.
3. **Verify Deployment Trigger:** Confirm the final Jenkins stage actually triggers a rollout (e.g., kubectl rollout restart or updating the task definition in ECS) rather than just pushing an artifact to a registry.
4. **Inspect Target Environment & Branch:** Verify the pipeline deployed to the intended cluster/namespace and checked out the right Git branch rather than a stale workspace cache.
5. **Check CDN/Reverse Proxy Cache:** Purge Cloudflare, CloudFront, or Nginx caches if static assets or API responses are cached at the edge.


### 3. How would you design an end-to-end CI/CD pipeline with automated testing, security scanning, approval gates and rollback?

flowchart TD
    A[Developer Push / PR] --> B[1. Source & Pre-Checks<br/>Git Checkout, Linting, Secret Scan]
    B --> C[2. Build & Unit Testing<br/>Compile, Unit Tests, Build Image with Git SHA]
    C --> D[3. Automated Security Gates<br/>SAST, Dependency Scan, Container Image Scan]
    D --> E{Critical CVEs Found?}
    E -- Yes --> F[Fail Build & Notify Team]
    E -- No --> G[4. Staging & Dynamic Validation<br/>Push Image, Deploy Staging, E2E & DAST Tests]
    G --> H[5. Manual Approval Gate<br/>Release Manager Sign-off]
    H --> I[6. Production Canary / Blue-Green<br/>Progressive Traffic Shift 10% -> 100%]
    I --> J{Health Check Breach?}
    J -- Yes --> K[Automated Rollback to Previous Revision]
    J -- No --> L[Production Deployment Complete]




### 4. How would you deploy the exact same Docker image across Dev, QA and Production environments?
1. **Build Once, Promote Many:** Build the Docker image once during the CI phase and tag it with an immutable identifier (e.g., app:git-sha-a1b2c3d).
2. **Externalize All Configuration:** Follow the Twelve-Factor App methodology: never bake environment variables, API URLs, or secrets into the Dockerfile. Inject them at runtime via Kubernetes ConfigMaps/Secrets, AWS Secrets Manager, or environment files.
3. **Promote via GitOps / Registry:** Push the verified image to a central registry (ECR/GHCR). Promote it across environments by updating only the image tag reference in each environment's deployment manifest (e.g., dev.yaml → qa.yaml → prod.yaml).


## Jenkins Q&A from ADP

## 15. Jenkins Pipeline Flow
Git → Build → Test → SonarQube → Docker Build → ECR → EKS Deployment

## 16. Jenkins Troubleshooting
- Check master logs
- Agent connectivity
- Plugin compatibility
- Credentials
- Network issues

## 20. Jenkins Best Practices
- Shared Libraries
- Pipeline as Code
- Secret Management
- Parallel Builds
- Versioned Artifacts

## 21. Shared Libraries
Reusable Jenkins pipeline code stored centrally.
