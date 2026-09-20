# Aviz AWS interview Questions(realtime interview questions):

## straight questions: 

1. Explain the difference between IAM Users, Groups, Roles, and Policies. When would you use each?

A. An IAM User is a permanent identity with long-term credentials - I create these only for real humans who need
console or CLI access. Groups are collections of users; I attach policies to groups (like Developers, DBAs, ReadOnly)
instead of individual users so permission management scales. Roles are identities with temporary credentials that get
assumed - I use them for EC2 instances, Lambda functions, cross-account access, and CI/CD pipelines so no long-
term keys are involved. Policies are the JSON documents that actually define permissions, and they attach to users,
groups, or roles. In my projects the rule is: humans get users in groups (or better, SSO), workloads always get roles, and
policies are customer-managed for version control.

2. What is the difference between identity-based policies and resource-based policies? Give a production example
for each.

A. An identity-based policy attaches to a user, group, or role and says what that identity can do. A resource-based policy
attaches to the resource itself and says who can access it. Production example of identity-based: my ECS task role has
a policy allowing dynamodb:GetItem and PutItem on one specific table. Resource-based example: an S3 bucket policy
on our central logging bucket that allows CloudTrail and ALB log delivery from all our accounts, plus a KMS key policy
allowing specific roles to decrypt. The key difference - resource-based policies can grant cross-account access
directly because they name the principal, while identity-based policies only work within the account that owns the
identity.

3. How does IAM policy evaluation logic work? Walk through the order - explicit deny, SCP, permission boundaries,
identity policies, and resource policies.

A. The evaluation always starts with an implicit deny - nothing is allowed by default. Then the order is: first, if there is an
explicit Deny anywhere (identity policy, resource policy, SCP, boundary), the request is denied immediately - deny
always wins. Second, SCPs from AWS Organizations must allow the action - if the SCP doesn't allow it, it fails
regardless of the identity policy. Third, if a permission boundary is set, the action must be allowed in both the boundary
AND the identity policy - the effective permission is the intersection. Fourth, either the identity-based policy or the
resource-based policy must explicitly allow the action (within the same account, either one is sufficient). If nothing
allows it, the implicit deny applies. I remember it as: explicit deny > SCP > boundary > an explicit allow somewhere.

4. What is the IAM policy evaluation flow when a user from Account A tries to access an S3 bucket in Account B?

A. Cross-account access requires an allow on both sides. In Account A, the user or role needs an identity-based policy
allowing the S3 actions on the bucket ARN in Account B. In Account B, the bucket policy must explicitly allow the
Account A principal (or the whole account) to perform those actions. If either side is missing, the request is denied.
Also, SCPs in both accounts must not block it, and if the bucket uses SSE-KMS, the principal also needs kms:Decrypt on
the key in Account B - the KMS key policy has to allow that too. That KMS piece is what I've seen break cross-account
access most often in real projects.

5. What are IAM Permission Boundaries and how do they differ from standard policies? When would you use them?

A. A permission boundary is an advanced feature where you attach a managed policy to a user or role that defines the
maximum permissions that identity can ever have. It doesn't grant anything by itself - the effective permission is the
intersection of the boundary and the identity policy. The classic use case, which we used in my project, is delegated
role creation: developers can create IAM roles for their Lambda and ECS workloads themselves, but an SCP forces
every role they create to carry our standard boundary policy. So even if a developer attaches AdministratorAccess to
their role, the role can only actually do what the boundary allows. Standard policies grant; boundaries cap.

6. Explain the difference between AWS-managed policies, customer-managed policies, and inline policies. What are
the pros and cons of each?

A. AWS-managed policies are created and maintained by AWS (like AmazonS3ReadOnlyAccess) - good for quick starts
and AWS keeps them updated for new services, but they're often broader than you want and you can't edit them.
Customer-managed policies are ones you write - they're reusable, versioned (up to 5 versions with rollback), and you
can manage them through Terraform or CloudFormation, which is what I do in production. Inline policies are
embedded directly in a single user or role - they can't be reused, are harder to audit, and get deleted with the identity.
My practice: customer-managed for everything standard, inline only for tightly-coupled one-off exceptions where I
deliberately want the policy to die with the role.

7. How do you implement least privilege access in an organization with 500+ developers across multiple AWS
accounts?

A. At that scale you cannot manage individual users - I'd approach it in layers. First, no IAM users at all: everyone comes
through IAM Identity Center federated to the corporate IdP, so access is group-based and centrally revoked. Second,
define permission sets per persona - developer, DBA, SRE, read-only - instead of per person, and map IdP groups to
accounts. Third, put guardrail SCPs at the OU level (deny leaving allowed regions, deny disabling CloudTrail, deny IAM
user creation). Fourth, use permission boundaries so teams can self-service roles for their workloads without
escalation. Fifth, continuously right-size: IAM Access Analyzer policy generation from CloudTrail and Access Advisor
last-accessed data to strip unused permissions. Least privilege at scale is a process, not a one-time setup.

8. What is the difference between AssumeRole, AssumeRoleWithSAML, and AssumeRoleWithWebIdentity? When is
each used?

A. All three are STS operations that return temporary credentials, but the caller differs. AssumeRole is used by an existing
AWS principal - a user or role - to switch into another role, typically for cross-account access or privilege separation;
it can be protected with ExternalId and MFA. AssumeRoleWithSAML is used for enterprise federation - the user
authenticates against a SAML IdP like Azure AD or Okta, and presents the SAML assertion to STS instead of AWS
credentials; this is how corporate SSO to the console works. AssumeRoleWithWebIdentity exchanges an OIDC token
for credentials - no AWS credentials needed to call it. That's the foundation of IRSA in EKS and GitHub Actions OIDC
deployments, which is how my pipelines authenticate to AWS without stored secrets.

9. Explain the concept of IAM Roles for Service Accounts (IRSA) in EKS. Why is it better than using node-level IAM
roles?

A. IRSA lets an individual Kubernetes service account map to an IAM role, using the EKS OIDC provider. The pod's service
account token is exchanged via AssumeRoleWithWebIdentity for temporary credentials scoped to that role. With node-
level roles, every pod on the node inherits the same permissions - so if one pod needs S3 write, all pods on that node
effectively get it, which breaks least privilege and blast-radius isolation. With IRSA, my payment service pod gets only
its DynamoDB permissions and my report service pod gets only its S3 permissions, even on the same node. The trust
policy conditions on the specific namespace and service account, and credentials are temporary and auto-rotated.
Newer alternative is EKS Pod Identity, but the principle is the same - identity per workload, not per node.

10. What is STS (Security Token Service)? How does it work with temporary credentials and what are its key API calls?

A. STS is the service that issues temporary, limited-lifetime credentials - access key, secret key, and a session token - so
you never need to distribute long-term keys. Sessions last from 15 minutes up to 12 hours depending on configuration.
Key API calls: AssumeRole (cross-account and role switching), AssumeRoleWithSAML (enterprise SSO),
AssumeRoleWithWebIdentity (OIDC - IRSA, GitHub Actions), GetSessionToken (mainly for MFA-protected API access
with an IAM user), GetCallerIdentity (the whoami of AWS - my first troubleshooting command), and
DecodeAuthorizationMessage for decoding encoded access-denied errors. Everything role-based in AWS - instance
profiles, Lambda execution roles, federation - is STS under the hood.

11. How do Service Control Policies (SCPs) work in AWS Organizations? Can an SCP grant permissions?

A. SCPs are organization-level guardrails applied to accounts or OUs in AWS Organizations. They never grant permissions
- they only define the maximum available permissions for an account. The actual grant still has to come from an
identity or resource policy inside the account. SCPs filter down: an action must be allowed at every level of the
hierarchy (root - OU -> account) to be usable, and they apply to everyone in the account including the account root
user. Typical guardrails I've implemented: deny regions outside our approved list, deny CloudTrail/Config tampering,
deny IAM user access-key creation, and deny deletion of logging buckets. One catch - SCPs don't affect the
management account or service-linked roles.

12. What happens when an IAM user has an explicit allow in their policy but an explicit deny in the SCP? Who wins?

A. The explicit deny in the SCP wins - deny always beats allow, at every level of evaluation. Actually, even without an
explicit deny, if the SCP simply doesn't allow the action, the user is blocked, because the effective permission is the
intersection of the SCP and the identity policy. The identity policy's allow only matters within the space the SCP
permits. This is exactly why organizations use SCPs as guardrails: no admin inside a member account can override
them, since they're controlled from the management account.

13. What is the purpose of aws:SourceIp, aws:RequestedRegion, and aws:PrincipalOrgID condition keys? Give a real-
world use case for each.

A. aws:SourceIp restricts by caller IP - real use case: our bucket policy for an internal reporting bucket denies access
unless the request comes from the office VPN CIDR ranges. aws:RequestedRegion controls which region the API call
targets - we used it in an SCP to keep all workloads in ap-south-1 and us-east-1 for compliance, while exempting
global services. aws:PrincipalOrgID checks that the calling principal belongs to your AWS Organization - the cleanest
way to secure a shared S3 bucket or KMS key: instead of listing 20 account IDs in the resource policy, one condition
"aws:PrincipalOrgID": "o-xxxx" allows the whole org and nothing outside it. That last one massively simplified our
central artifact bucket policy.

14. How do you secure the root account in a production AWS environment? List at least 6 best practices.

A. My checklist: (1) Enable hardware MFA on root - not just virtual. (2) Delete any root access keys - root should never
have programmatic access. (3) Use root only for the handful of tasks that require it (account closure, some billing
settings) and log in through a monitored break-glass procedure. (4) Set a strong unique password stored in a controlled
vault with dual custody. (5) Create a CloudWatch/EventBridge alert on any root login or root API activity from CloudTrail
- this should page someone. (6) Set account security contacts and verify the root email is a distribution list owned by
the org, not an individual. (7) At org level, use SCPs to restrict what root in member accounts can do, and prefer
Organizations-created accounts where root has no password until reset.

15. What is IAM Access Analyzer and how does it help with security posture management?

A. Access Analyzer continuously analyzes resource policies - S3 buckets, IAM roles, KMS keys, Lambda, SQS, secrets -
and flags any resource shared outside your zone of trust (your account or organization). It's based on formal reasoning
over the policies, not traffic. In practice I use it for three things: detecting unintended external access (a bucket policy
or role trust that allows an unknown account - finding shows up immediately), generating least-privilege policies from
actual CloudTrail activity - you point it at a role's history and it drafts the policy covering only what was really used,
and validating policies at author time with its policy checks for security warnings and errors. The unused-access
analyzer also reports roles and permissions not used in N days, which feeds our quarterly access reviews.

# TCS interview Questions


### 1. What is the difference between ALB & NLB?
A. **Application Load Balancer:**An Application Load Balancer (ALB) operates at Layer 7 (HTTP/HTTPS) and routes traffic based on application content such as Path-based, and also routes host-based, query string parameters(in ALB rule condition if we provide v2 it will route traffic to v2 or if it's ). it is slow(due to layer inspection).
it uses Dynamic IP(use DNS name) and don't use static IP.

ALB uses in web applications, e-commerece applications.

**Network Load Balancer:**. A Network Load Balancer (NLB) operates at Layer 4 (TCP/UDP) and routes raw traffic based strictly on IP addresses and ports without inspecting content. it uses static ip(an ip address doesn't change in AWS have EIP). it's extremely fast. when you need extreme performance, require ultra-low latency, are handling non-HTTP traffic.

NLb uses in gaming, iot Devices and trading to handle millions requests and sudden spikes in financial transaction traffic during market openings.

### 2. if EC2 instance is unreachable then how would you troubleshoot it?
1. i will check the EC2 instance is up and running.
2. i will verify SG's port 22 open for ssh and port 3389 open for RDP(windows). then i will verify the NACL's allowing the inbound and outbound traffic allowing.
3. i will the check the permissions of ssh private key and set the permissions 0664. then i will connect the ec2 through ec2 serial console and checks the sshd service is running.
4. i will check the instance screenshot in ec2 console by clicking *Actions > Monitor and troubleshoot > Get instance screenshot* to see if it's a blue screen or kernel panic and checks the system logs by clicking on *get system logs*.
5. i will also check the cloudwatch logs and metrics to see the cpu and other details.

### 3. how will you check if app performance is slow?
1. First, I check the Load Balancer metrics like ALB response times if it's high, it confirms the issue is on our backend.
2. then I also look at the HTTP status codes(like 503, 500 errors).
3. next i will check the load on server using cloudwatch metrics like cpu and memroy utilization. if app code is consuming more cpu or any memory leaks in the code i will collect the thread dump and heap dump and shares those with developers.
4. if server looks fine i will trace the network between applications using tracing tools like jagar or AWS x-ray to identify the bottelnecks, and also checks the Garbage Collection (GC) pauses in Java, or Event Loop delay in Node.js, which can cause intermittent freezing.
5. then i check the database layer I look at Active Connection Counts to see if the app is blocking while waiting for a DB connection, then checks Slow Query Logs and Database CPU to identify unindexed tables or heavy joins that are locking up rows.
in RDS i will the read replicas to see there is any sync lag between original RDS and RDS read replicas due to high traffic/cpu in my main database the sync is delayed.
6. Once the immediate issue is resolved, I establish long-term fixes. This includes setting up CloudWatch Alarms on p95/p99 response times, configuring autoscaling policies based on request counts, and ensuring we have robust caching layers like Redis or CloudFront to protect our database from repetitive hits."

### 4. you found a process it is consuming 40% to 50% of memory what will you do and what will be approach?
1. i will check which process is consuming more memory using *top and ps* commands then i will check blastradius(scope of impact) of that process before terminating.
2. then i will check if it's normal behaviour for heavy application or it's a memory leak.
3. i will check memeory allocation for that process using VIRT(VSZ) virtual memory and RES memory. RES tells exactly how much physical RAM process holding now.
4. I’ll run free -h to see how much total buffer/cache and free RAM remains on the host. If the system still has 30%+ free memory and isn't swapping(if it's swapping server is under heavy RAM pressure), the instance is not in immediate danger of crashing.
5. If it is a Database (like PostgreSQL/MySQL) or a Java app, 50% memory consumption might be completely normal due to pre-allocated buffers i will confirm this using historical metrics(cloudwatch metrics) in cloud watch If memory consumption is a flat line at 50%, it's normal. If it is a steady, upward staircase over hours or days, it is a memory leak.
6. if it's emergency i will restart service/server(temporary solution). if it's a memory leak i will share the thread and heap dumps to developer team to resolve the memory leak till then i will schedule graceful daily/weekly service restarts until solution deployed to prod.
7. if it's not a memory leak it's normal i will vertically scale the server.

**Swapping:** Swapping means the operating system moves less-used data from RAM to disk space called swap(swap memory) when RAM is getting full. This frees up memory for active processes, but it is much slower than RAM, so performance can drop significantly.

### 5.How restarting a service or server resolves the high cpu or memory usage problems?
A. **memrory issues:**
1. It Completely Wipes Memory Leaks
2. It Clears Fragmented Memory Pools:Long-running processes frequently allocate and deallocate memory blocks, leaving tiny gaps of unallocated space across the RAM sticks. Even if total free memory looks sufficient, the OS might fail to find a large, continuous block of memory for new requests, causing allocation errors or slowdowns.
3. It Instantly Empties the Swap Space

**CPU Issues:**
1. It Kills Garbage Collection Loops
2. It Terminates Infinite Code Loops and Deadlocks
3. It Clears Blocked Network and File Descriptors

### 6. what is cost forecasting how it helps in AWS?
A. Cost forecasting is the process of using historical spending data and machine learning algorithms to predict your future cloud expenditures over a specific period (e.g., the next month, quarter, or year).

in AWS we have AWS Cost Explorer that has built-in forecasting capabilities.

### 7. how to know a instance is on-demand or spot or reserved?
A. we can know using console and aws cli. in ec2 console select the instance in details tab look lifecycle it is normal on-demand instnace if it is spot it's spot instnace. but for reserved or other type instance it shows normal only to know instnace is reserved or not look into billing console(AWS cost explorer) same goes for cli method also(only identifies spot or on-demand).
#### cli method:
```sh
aws ec2 describe-instances --instance-ids i-1234567890abcdef0 --query "Reservations[*].Instances[*].[InstanceId,InstanceLifecycle]" --output text
```
![alt text](.images/lifecycle.png)

### 8. My ec2 instances are using EFS autoscaling group will scale the instances every time when adding new ec2 instance we need to mount the EFS id don't want to do this how would you resolve this issue?
A. it's difficult manaually login to every new instnace and mount EFS. so we use *GoldenAMI* we will create a ami with EFS already mount to desired mount point(dir like /var/www/html) and in Autoscaling group launch template we will use this ami to create instances.

### 9. When you create a new vpc, what are the components you get defualtly?
A. RouteTable, NACL, DefaultSecurityGroup

### 10. A user(iam user)is not able to access the objects in the s3 bucket has s3 full access policy, bucket has no policies attached, no permissions boundary set for tha user but still user is not able to access the bucket what could be the reason?
A. issue might be with kms key policy if the bucket is encrypted with kms key. if the user doesn't have kms key permissions then user can't access the objects in the bucket. goto kms key policy and add the user arn to the key policy(or add the user in key users section in key policy tab).

the question might be with different way also first trouble shoot like below:
1. check the bucket policy and make sure there is no explicit deny for that user.
2. check the bucket ACL and make sure the user is not denied access.
3. check the IAM policy attached to the user and make sure it allows access to the bucket and there is no deny statement.
4. check the permissions boundary for the user and make sure it allows access to the bucket.
5. check the KMS key policy if the bucket is encrypted with a KMS key and make sure the user has access to the key.

Note: ask the interviewer what kind of error he is encountering we can easily troubleshoot using the error.

11. your AWS account is compromised how will you recover it?
A. Disable credentials, Reset the password, Enable MFA..!!
--> Did he created any other user.? i will check using cloud trail.
--> What are all the other activities this guy performed.?
--> Block network Traffic at NACLs. (VPC FlowLogs, Login--> Cloudtrail)
--> Incident investigation.


## Infosys interview questions

### 1. i have 300 aws accounts how can i check the vulnerabilities in these accounts using single dashboard?
A. AWS Inspector, guard duty and AWS Security HUB.

**AWS Inspector:** An automated vulnerability management service that inspects the inside of your workloads (such as EC2 instances, ECR container images, and Lambda functions) for software flaws and insecure configurations. it looks our resourse *inside out*(inspects internal components—packages, OS libraries, application dependencies detect those vulnerabilities before an exploit happens.)

It detects Outdated software packages, known software vulnerabilities (CVEs), and unintended network exposure.

**EX:** EC2 instance i-12345 is running an outdated version of OpenSSL that is vulnerable to data leaks

**AWS Guard Duty:** A managed threat detection service looks at your environment from the *outside in* continuously monitors your  AWS account activity, logs, and data patterns for malicious or unauthorized behavior. it monitors VPC Flow Logs, CloudTrail management/data events, DNS logs, EKS audit logs, and RDS login attempts etc.

**EX:** EC2 instance i-12345 is actively sending traffic to a known Bitcoin mining pool.

**AWS Security Hub:**AWS Security Hub is a cloud security posture management (CSPM) service that acts as a central dashboard for security findings from AWS inspector, guard duty and other AWS and third-party tools.

i will use both Guard duty and inspector to detect vulnerabilities and malicious activity in AWS accounts and then AWS security Hub aggeregates these findings via AWS organisations(Without this,we manually generate IAM roles, trade API keys, and accept 300 individual cross-account invitations it is a heavy process that takes weeks) and that data will be visible in a single dashboard.