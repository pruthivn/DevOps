# Kubernetes

This directory contains Kubernetes-related DevOps interview questions and resources.

## 1. What happens in k8's when you run kubectl apply?
![alt text](.images/kubectl_apply.png)

## 2. why do you prefer helm charts over Kubernetes YAML manifests?
A. Helm charts are preferred over plain Kubernetes YAML manifests because Helm provides templating, reusable charts, environment-specific configuration through values.yaml, and easier application upgrades and rollbacks, which simplifies managing deployments across environments like Dev, UAT, and Production.

## 3.How do you handle environment-specific configuration in Helm charts?
A. In Helm, different environments such as Dev, UAT, and Production are usually differentiated using separate values files like values-dev.yaml, values-uat.yaml, and values-prod.yaml, Each file contains environment-specific configurations such as replica count, resource limits, and environment variables. During deployment, the CI/CD pipeline selects the appropriate values file and deploys the application using the Helm command with that file.

## 4. How do you handle failure in deployment?
1. i deployment fails i immediaetly rollback to the previous stable app version and inform the dev teams and client about the issue.
2. after that i wll check the logs and monitioring dashboards to identify the issue.
3. then i will investigate the rootcause issue by reviewing recent code changes, Merge requests and deployment scripts to identify issues like it is code related issue or config issue or network issue or infra related issue.
4. after identifying the issue i will solve that like if it is code realted communicate with dev teams or if it is network issue i will resolve that network issue.
5. then i will test it dev or locally and redeploy it into prod.
6. then i will document this issue like cause of issue, what actions taken to resolve this.
7. i will improve the CI/CD pipeline to catch issues eariler and implements health checks and automatic rollbacks.
8. i will configure alerts in montioring toools to detect and give alert when issues occur.

## 5. what is p50, p95 and p99?
A. P95, p50 and P99 latencies are percentiles used to measure the performance of a system from the perspective of your worst-experienced user requests.

EX: If you collect 100 user requests and sort them in a line from the fastest response time to the slowest:

P50 = 0.50 × 100 = 50th request.
P95 = 0.95 × 100 = 95th request.
P99 = 0.99 × 100 = 99th request.

P50 (Median): This is the exact middle request (the 50th request). If your P50 is 100ms, it means 50% of your users experienced a speed of 100ms or faster.

P95 (95th Percentile): This is the 95th request in line. If your P95 is 500ms, it means 95% of users experienced speeds under 500ms(means 95th request taking 500 ms to complete), while the remaining 5% experienced speeds worse than 500ms.

P99 (99th Percentile): This is the 99th request in line—the absolute worst-case scenario. If your P99 is 2 seconds (means 99th request taking 2 sec to complete), it means 99% of your users had speeds under 2 seconds, but 1% of your users waited longer than 2 full seconds.

If you only reported the average latency (for example 200ms), you’d think your system is blazing fast.

But in reality, 1% users every minute are waiting 2+ seconds — and they’re the ones filing support tickets or abandoning your app.

that's where P95 and P99 are comes into picture to identify those 1% requests. 

**Resolution:**

Thread pools, connection limits, caches, retries, and circuit breakers all exist because of P95/P99 realities.

### 6. explain EKS upgrade?
1. first read the release notes of the new version carefully to how new changes impact our application.
2. take the backup of etcd server using *etcdctl* command.
3. before updating control plane check the kubectl version is same in control plane and in node for example if control plane is 1.28 and node has 1.27 we must update the version in node to 1.28 before the updating the control plane to 1.29 or 1.30.

**Note:** if we didn't upgrade nodes kubectl version to control plane version AWS will completely block your control plane upgrade.
4. then we can upgrade the EKS cluster through AWS console or through aws cli using below command
```sh 
aws eks update-cluster-version --name <cluster-name> \
  --kubernetes-version <version-number> --region <region-code>
```
5. need to upgrade the one node at a time for availabilty of applications. first cordon(makes node unschedulable) the node using *kubectl cordon node_name* and then drain the node using *kubectl drain node_name* it safely evict the pods(pods schedule on other nodes for high availablility). upgrade the node using below command
```sh 
eksctl upgrade nodegroup \
  --name=node-group-name \
  --cluster=my-cluster \
  --region=region-code
```
**Note:** if you see the error while updating or draining the node it is mostly due to Pod Disruotion Budget use *--force-upgrade* with eksctl command.
6. then uncordon the node using *kubectl uncordon*
7. repeat the same for every node.

### 7. what is the hardest/recnt k8's issue you encountered?
A.**Kuburnetes API Server Latency:**
1. we started receiving alerts from Prometheus/cloudwatch that application response time had increased significantly.
initially,i assumed the app was slow, but the pods are healthy. i thought issue is with nodes but nodes have sufficient memory and CPU.
2. Kubectl commands taking 20-30 seconds to execute, HPA not working, Deployments stuck, controller lagging it indicates problem with control plane.
3. Instead of immediately restarting components, I first wanted to determine whether the K8's API Server was healthy.
I ran:
```sh
kubectil get --raw'=/readyz?verbose'
```
This endpoint checks the readiness of the API Server and verifies that all its internal dependencies-such as etcd, admission controllers, authentication, and authorization are functioning correctly.
If the API Server is unhealthy, this command usually reports which component is failing.
4. Next, I checked API Server metrics(because every K8's operation goes through API server).I looked for, Request latency, Total API requests, Active watch connections, Request queue length, Error rate, Control Plane CPU, Memory usage, etcd response time. At this stage, I noticed that the API Server CPU utilization was consistently above 90%.
5. At this point, I wanted to identify who was generating so many API requests. The API Server logs showed repeated requests coming from one controller. After reviewing its logs, we found that the controller was continuously polling the API Server every second instead of relying on Kubernetes watch events.

| Efficient: Watch  | Inefficient: Polling |
|---|---|
| Watch Deployment | Every second |
| → Wait for changes | → GET Deployments |
| → Process only changed objects | → GET Services |
|  | → GET Pods | 
|  | → Repeat forever |

This unnecessary polling generated thousands of API requests every minute.

6. configured rate limit for API server(previusly controller sent 1000 requests/min now 50 request/min), added cliend-side cache(now controller read the info from cache instead of querying API server), reduced watch connections by removing the duplicate watchers, added dashboards for monitoring API server latency, request rate, API server cpu, etcd latency, watch connection, error rate etc., using these metrics we detect issues early.

### 8. What is watch request in K8's?
A. a specialized HTTP long-polling mechanism that allows clients to receive real-time, streaming updates about resource changes (creates, updates, and deletes) without continuously polling(requesting) the API server. controllers(deployment, replicaset controllers) use the watch api instead of requesting the api server for resource changes.

This is how built in controllers like Deployment and replica sets operate in K8's.

### 8. Pods are Running, Service exists, but the app is not accessible, *kubectl get endpoints* shows None how can you solve this issue?
A. if *kubectl get endpoints* shows None the service isn't linked any pod. compare the service selectors with pod lables, maybe label mismatch is the comman cause and ensure pod lables are matching the service selector.

check the pods are ready or not using *kubectl get pods* if pods are not ready readiness probe is failed if it failed it removes pods from service.

### 9. What is the Difference between CreateContainerConfigError and CreateContainerError?
A. **CreateContainerError:** This error is commonly due to *cmd or Entrypiont* errors(like invalid commands in *cmd*), invalid mount points, invalid ports or configuring already using ports. this error occur during the container creation.

Make sure commands valid in cmd or entrypoint,configured volumemount paths, using right ports then restart the deployment to resolve this issue.

**CreateContainerConfigError:** This error is commonly due to missing config maps, secrets, env vars, volume mounts mismatch(path is not correctly configured) and incorrect key values in config maps or secrets. this error occurs before the container runtime attempts to launch the container process.

Make sure config maps, secrets, env vars, volume mount are correctly configured and then restart the deployment to resolve this issue.

### 10. What is *back-off restarting failed container* error in K8's?
A. The "Back-off restarting failed container" error (commonly visible as *CrashLoopBackOff* k8's logs) means that your container is repeatedly crashing immediately after starting, and Kubernetes is delaying its next restart to protect cluster resources.

the issue occurs due misconfigurations in config maps. secrets and service account misconfiguration, resource issues, incorrect env vars, missing app dependencies, failing livness probes and issues with third party services(pod will crash if it depends on external services that have problems with DNS, database, or API.)

**Backoff:** means When a container keeps failing to start, k8's doesn’t keep restarting it immediately. so it uses *Backoff* mechanism which increases the wait time before each restart attempt. 

Ex: the wait time typically grows exponentially:
10s → 20s → 40s → 80s → … (up to 5 minutes)

This mechanism prevents that restarting container(uses more cpu and memory while restarts) consume more cluster resources.

### 11. What is init container?
A. **init containers:** it's special containers that run before the main application containers in a pod start. it is used to verify external dependencies(like a database or external API becomes reachable), Managing Configuration(Downloading secrets, certificates, config files into shared volume), run database migrations. the main container only when init containers finishes successfully. init containers must exit with a status code of 0.

If you define multiple init containers, they run one at a time in the exact order specified.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-init-pod
  labels:
    app: api
spec:
  # Shared volume to pass migration logs or configurations
  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  # FIRST INIT CONTAINER: Check network dependency
  - name: wait-for-database
    image: busybox:1.36
    command: ['sh', '-c', 'until nc -z db-service 5432; do echo waiting for db; sleep 2; done']

  # SECOND INIT CONTAINER: Run schema updates
  - name: run-migrations
    image: migrate/migrate:v4.15.2
    command: ['migrate', '-path', '/migrations', '-database', 'postgres://db-service:5432/mydb', 'up']
    volumeMounts:
    - name: shared-data
      mountPath: /var/log/migrations

  # MAIN APPLICATION CONTAINER
  containers:
  - name: api-service
    image: my-api-app:v2.1
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-data
      mountPath: /app/logs
```
### 12. K8's secret exists but our app not able access it?
1. Check the secrets exists in correct *Namespace* using describe command, and check the service account has access the read the secrets(if not ensure service account has RBAC role to get the secrets).
2. check the secrets are correctly configured and referrenced in yaml manifests(like invalid volumemounts  and spelling minstakes, key value errors).
3. if secrets configured as env vars new secret changes will not be reflected in container until pod restarts, so configure secrets as volumes.

### 13. Difference between ImagePullBackOff or ErrImagePull and ErrImageNeverPull?
A. **ErrImageNeverPull:** this error occurs when we configured imagePullPolicy: Never(means k8's can't able pull the image from registry) in yaml manifest. if the image is no available pod crash's immediately. change the imagePullPolicy: always(it always pull the image from registry).

imagePullPolicy: IfNotPresent(Uses the local image if it exists; downloads registry it if it doesn't.)

**ImagePullBackOff or ErrImagePull**: this error occurs when image name or registry path not configured correctly, Auth issue with Registry, network issue(like fireall rules/proxy blocking registry access or registry not available). check the image configured correctly using describe command then check the registry access(firewall/proxy rules allowing or not).

### 14. pod stays in pending state event shows failed scheduling how will you resolve this issue?
1. use describe command to check the exact reason scheduling error. check the cluster has sufficient resource using *kubectl top nodes* insufficient resource are common cause. Check nodes nodes are healthy using *kubectl get nodes.*
2. Check the Node rules like Taint & toleraions, node-affinity, nodeselector are blocking the pod. take the confirmation to change the pod(tolerations) or node rules it not possible create a new node. use the cluster auto-scaler to scale k8's cluster.

### 15. Difference between Node Affinity, Node selector , Taint & tolerations?
A. if Node Affinity, Node selector are configured nodes attracts the pods but Taint and tolerations are configured nodes repel the pods. we cannot use taints to force a pod onto a specific node, is we use node selector pod lable not match pod stuck in peding state forever.

Node affinity have Advanced matching rules (like podAffinity) require the K8's scheduler to evaluate every node against every pod. This slows down cluster performance in large environments.

1. **Node selector:** A Node Selector is a simple key-value pair match. we will add lables to the nodes it pods matches this label pods will be schedule on this nodes.

```sh 
kubectl label nodes worker-node-01 disktype=ssd
```
```yaml
spec:
  nodeSelector:  # The scheduler evaluates this block before placing the Pod
    storage-type: ssd   # Must exactly match the key and value of the node label
  containers:
```
**Drawback:** If no nodes match the exact label, the pod will get stuck in a Pending state forever. It cannot fallback to other nodes. to overcome this we have Node affinity.

**Note:** for Cloud clusters we don't need to label the nodes( Cloud Controller Manager instantly reads the node's properties and applies standard K8's labels automatically) for on premisses we need to lable the nodes.

2. **Node Affinity:** we have Hard and Soft rules

**HardRule(requiredDuringSchedulingIgnoreDuringExecution):** Pod must run on a matching node, if no match found, it stays pending.

**SoftRule(preferredDuringSchedulingIgnoreDuringExecution):** Pod prefer to run matching nodes if no node founds it will run on other nodes.

**Drawback:** IgnoreDuringExecution : Running pods won't be evicted if labels change. if we remove the node label after running the pods with this label "RequiredDuringSchedulingIgnoredDuringExecution" the pods still run in those nodes.

to overcome this k8's introduced  *RequiredDuringExecution* but it is not widely used.

**Taint & tolerations:** we will add taint to nodes and tolerations to pods if pod toleration matches to node taint pod will be scheduled on that node otherwise it will repel the pods.

**Drawback:** if a node has no taint pods with tolerations will still run on that Node.

### 16. Pod is created but it stays in container creating state?
1. Check the PVC is bound, Volumes are mounted(If your Pod uses a persistent volume, the new node cannot start the container until the previous node safely releases its lock on the physical hard drive.), storage is available storages issues commonly cause of this issue.
2. then check secrets & configs maps are configured correctly, check the network issues(like network plugin pods are running kube-system namespace)

### 17. PVC status is showing pending how can you solve this issue?
1. Check the requested storage class exists and set correctly.
2. check matching PVC available or not and check if any error in dynamic provisioning(in cloud any errors while creating volumes dynamically).

### 18. your app can't able connect with other service the error show DNS lookup failed how will solve this issue?
1. verify DNS using *kubectl exec -it pod_name nslookup kubernetes.default* command, if DNS fails pod can't resolves the service names.
2. Verify CoreDNS pods are up and running using *kubectl get pods -n kubesystem*. if not running check the logs for errors.
3. then check the /etc/resolv.conf file, network policies ensure pod can communicate with core DNS.

### 19. What is CRD(Custom Resource Definition) in K8's?
A. **CRD(Custom Resource Definition):** a feature in k8's that lets you extend the Kubernetes API by creating our own custom object types.

By default, Kubernetes only understands built-in objects like *Pods, Services, and Deployments*. Applying a CRD teaches Kubernetes a brand-new object type—such as a Database, Backup, or SSLWithCert—allowing you to manage it using standard kubectl commands.

we need two things CRD file(contains new object type named *MyDatabase*, and here are the fields it is allowed to have) and Operator(Custom controller) watches our custom resource and takes real-world action.

**UseCases:**
Cert-Manager: Uses CRDs like Certificate and Issuer to automate SSL certificates.

ArgoCD: Uses CRDs like Application and AppProject to manage GitOps deployments.

Istio / Linkerd: Use CRDs like VirtualService or Gateway to route service mesh network traffic.

### 20. Liveness probe is failing and pod keeps on restarting how would you dolve this issue?
1. i will inspect the pod using describe command to check the error is secrets or config maps, invaild volume mounts or service account, invalid image name/tags.
2. then i will verify the probe configuration like path, port, initialDelaySeconds, Period seconds, failurethreshold. and i will exec the pod use curl on healthy endpoint configure the healthy path(healthy endpoint).
3. then i will check the pod logs any issue from app side.

## Kubernetes / EKS Q&A from ADP 

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
### 3. Server looks healthy and application showing healthy in Argocd but client getting 500 error?
1. first check the logs(kubectl pod logs or kubectl describe podA) 
2. verify connectivity to dependencies
3. validate secrets/configmaps
4. check ingress configurations like ingress controller is installed,ingress is watching by ingress controller, ingress class name is defined in ingress.

it might be issue with database connectivity database is not ready to serve traffic by adding readiness probe(it will restart the app it is not ready to serve the traffic) we can solve this issue.

ArgoCD tells you that your 𝗞𝘂𝗯𝗲𝗿𝗻𝗲𝘁𝗲𝘀 𝗿𝗲𝘀𝗼𝘂𝗿𝗰𝗲𝘀 𝗮𝗿𝗲 𝗶𝗻 𝘁𝗵𝗲 𝗱𝗲𝘀𝗶𝗿𝗲𝗱 𝘀𝘁𝗮𝘁𝗲(like pods running). It does NOT check:
- Application logic
- External dependencies
- Database connectivity
- Third-party services

ArgoCD can be green while but your app is broken.

### 4. Think of that customer reports your application is slow. No alerts are firing. So, where do we begin?
1. First, I isolate the scope by checking CDN and Load Balancer metrics to see if the latency is global or isolated to a specific region or feature.
2. then i will verify entire app is slow or is it a specific high-overhead action, like generating a report from app or proccessing a checkout.
3. i will look at P95 and P99 latency metrics in cloudwatch or grafana dashboards(we can als see in new relic) if any issue with this metrics i will use tracing tools like jaeger to identify the where the slowness like connectivity with database, an un-indexed database query, or a slow third-party API call.
4. then i will check the load on server like application is consuming more memory/cpu using uptime, sar, top commands, then will check file descriptor limits if app hits the heavy traffic file descriptor limits reaches and it causes to network packets to stall(paused, dropped).

### 4. Can the pod be OOM killed even without memory limit?
A. yes without defining the resource quota and limit pod can oom killed because if the node runs out of memory it evicate the pods according Qos(quality of service).

### 5. A node has a 32 GB RAM free But pod still reports OOM killed. Could you please explain what could be the reason for this?
A. in pod we configured resuorce limits if the pod reaches this limit it will automatically OOM killed.


## TCS Interview questions
### 1. what is the difference between persistent volume and persistent volume claim what role does it play in stateful applications?
A. **PV(persistent volume):**Persistent Volume (PV) is the actual storage resource provisioned in your cluster. it's scope is cluster level.

**PVC(Persistent Volume Claim):**while a Persistent Volume Claim (PVC) is a developer's request for that storage. it's scope is namespace level.

stateful applications need to save status and session info we can achieve this using PV and PVC. State-keeping apps like databases (MySQL, PostgreSQL) require data to remain intact if a container crashes, restarts, or moves to a different node. PVs and PVCs ensure the data lives longer than the Pod.

**Note:**If there is no PV to match the PVC, the StorageClass dynamically creates a PV and binds it to the PVC.

### 2. You need to ensure a specific pod is remain operational how to make sure the pod is always running?
1. using Deployments and provides desired replica counts.
2. config *restartpolicy: always* Forces the container to restart immediately if application process crashes.
3. using linvenss probes to restart stuck containers and Readiness Probes to stop traffic to the Pod by removing fro service if it becomes temporarily overloaded.
4. Adding Resource request and limits to Place the Pod in the Guaranteed Quality of Service (QoS) class.
5. Applying Pod Anti-Affinity it will prevents pods matching specific labels from being placed on the same node so all pods are distributed across the nodes.
6. creating PDB.