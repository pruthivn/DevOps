# Aviz AWS interview Questions(realtime interview questions):

---

### 1. Cross-Account Access Incident: Your application in Account A (Production) needs to read objects from an S3 bucket in Account B (Data Lake). The developer created an IAM role in Account A with S3 read permissions, but the application still gets AccessDenied. What could be wrong and how would you troubleshoot this?

A. Classic cross-account setup issue — I'd check both sides of the handshake. First, does the bucket policy in Account B explicitly allow the role from Account A? An identity policy in A alone is not enough cross-account. Second, is the application actually using that role — I'd run sts get-caller-identity from the instance to confirm the principal. Third, if the bucket uses SSE-KMS: the role needs kms:Decrypt and the key policy in Account B must allow it — this is the most common silent killer; you get s3 AccessDenied but the real issue is KMS. Fourth, check for blockers: an SCP on either account, a permission boundary on the role, an explicit deny with conditions (aws:SourceVpce, SecureTransport), S3 Block Public Access is irrelevant here but a VPC endpoint policy could restrict which buckets are reachable. CloudTrail on the S3 data events or the encoded authorization message tells you exactly which policy denied it.

---

### 2. Leaked Credentials: You receive an AWS Abuse notification that your IAM access keys have been exposed on a public GitHub repository. Walk through your complete incident response procedure, step by step.

A. Speed matters here because attackers scan GitHub within minutes. My steps: (1) Immediately deactivate the exposed key — deactivate, don't delete yet, so I preserve evidence and can roll back if something legitimate breaks. (2) Create a new key for the workload if it's genuinely needed, update it in the secret store, then delete the old one after validation. (3) Check CloudTrail for all activity from that key since exposure — look for unusual API calls, region activity we don't use, iam:* calls, ec2:RunInstances (crypto mining), s3 data access. (4) Check for persistence: new users, new keys, new roles, modified trust policies, Lambda backdoors. (5) Contact AWS Support if there's evidence of abuse; check billing for anomalies. (6) Purge the secret from git history (BFG/filter-repo), rotate anything else in that repo. (7) Post-incident: add git-secrets/trufflehog pre-commit hooks and pipeline scanning, move the workload from access keys to roles or OIDC so this class of incident disappears, and enable GuardDuty which detects exposed-credential usage patterns.

---

### 3. Permission Boundary Conflict: A junior DevOps engineer created a Lambda function with an execution role that has AdministratorAccess. Your organization uses Permission Boundaries. The Lambda function cannot write to DynamoDB even though the role policy allows it. What is happening and how do you resolve it?

A. This is the permission boundary doing exactly its job. The effective permissions of that Lambda execution role are the intersection of its identity policy (AdministratorAccess) and the attached boundary — and the boundary evidently doesn't include dynamodb:PutItem or write actions. AdministratorAccess inside the role is meaningless beyond what the boundary caps. To resolve it properly, I would not remove the boundary — I'd fix the role the right way: replace AdministratorAccess with a scoped policy for what the function actually needs (DynamoDB write on that table, logs, whatever else), and if DynamoDB write is legitimately required but missing from the boundary, get the boundary policy updated through the platform/security team's change process. This is also a coaching moment for the junior engineer on why admin-on-Lambda is never acceptable.

---

### 4. Federation Failure: Your company uses SAML-based federation with Azure AD for AWS Console access. On Monday morning, 200 users report they cannot log in to AWS. What are the possible causes and how would you diagnose this?

A. Since it's 200 users at once, it's systemic, not individual. My diagnosis order: (1) Check whether anything changed over the weekend — most Monday-morning outages are Friday changes. (2) The most common cause: the SAML signing certificate on the IdP side expired or was rotated without updating the IAM identity provider metadata in AWS — verify certificate validity and re-upload metadata. (3) Check Azure AD side: is the enterprise application healthy, did conditional-access policies change, is the SSO URL right. (4) Verify the IAM identity provider and role trust policies weren't modified — CloudTrail on iam:UpdateSAMLProvider or role trust changes. (5) Check clock skew and assertion validity windows. (6) Look at the actual SAML response with a browser SAML tracer — the error in the assertion (audience mismatch, missing Role attribute, wrong ARN format) usually pinpoints it. Meanwhile, use the break-glass IAM admin user to keep operations running.

---

### 5. Privilege Escalation Detection: CloudTrail shows that a developer’s IAM user performed iam:CreateRole, iam:AttachRolePolicy, and sts:AssumeRole in sequence within 5 minutes. What likely happened here? How would you prevent this in the future?

A. That sequence is a textbook privilege-escalation pattern: the developer created a new role, attached a powerful policy to it, and then assumed it — effectively granting themselves permissions beyond their own identity. If their user had iam:CreateRole and iam:AttachRolePolicy without constraints, they could mint an admin role with a trust policy pointing at themselves. First I'd treat it as an incident: check what the assumed role actually did, suspend the user's access, and review with them — it may also be an innocent but non-compliant workflow. Prevention: never grant broad iam:* to developers; require permission boundaries on any role they create (enforced via SCP condition on iam:PermissionsBoundary); restrict iam:PassRole and role trust policies; use IAM Access Analyzer and GuardDuty findings; and alert on exactly this CloudTrail sequence via EventBridge rules. Role creation for workloads should go through IaC with review, not console clicks.

---

### 6. Multi-Account SCP Challenge: Your organization has 15 AWS accounts under AWS Organizations. The security team wants to ensure no one — not even account admins — can launch resources outside ap-south-1 and us-east-1. How would you implement this while allowing global services like IAM, CloudFront, and Route 53 to function?

A. This is an SCP use case. I'd attach a deny-based SCP at the root or OU level: Deny all actions when aws:RequestedRegion is not ap-south-1 or us-east-1, with a NotAction exemption list for global services — IAM, Organizations, Route 53, CloudFront, WAF, Support, Trusted Advisor, STS, and billing/Cost Explorer actions — because those are served out of us-east-1/global endpoints and would otherwise break. Deny-list SCPs are better than allow-lists here because they survive AWS adding new services. I'd test it on a sandbox OU first, watch CloudTrail for unexpected denials, then roll it out per OU. Since SCPs apply to all principals including account admins, this meets the "not even admins" requirement — only the management account is outside SCP enforcement, which is why nobody works in the management account.

---

### 7. CI/CD Pipeline Permissions: Your Jenkins pipeline running on an EC2 instance needs to deploy to ECS, push images to ECR, update parameters in Systems Manager, and invalidate CloudFront distributions. How would you design the IAM permissions for this pipeline? What is the wrong approach many teams take?

A. The wrong approach — which I've genuinely seen — is creating an IAM user with AdministratorAccess and pasting its keys into Jenkins credentials. Long-term admin keys on a build server are the single biggest blast radius you can create. My design: the Jenkins EC2 instance gets an instance profile role with a scoped policy — ecr:GetAuthorizationToken plus push/pull on specific repos, ecs:UpdateService/RegisterTaskDefinition on specific clusters/services, ssm:GetParameter/PutParameter on our parameter path prefix, cloudfront:CreateInvalidation on the specific distribution — each with resource ARNs, not *. iam:PassRole limited to the exact task execution roles, with a Condition on iam:PassedToService being ecs-tasks.amazonaws.com. If different teams share Jenkins, per-pipeline deployment roles that the base role can assume, so permissions are isolated per pipeline. No stored AWS keys anywhere, credentials are temporary and auto-rotated by the instance profile.

---

### 8. Temporary Access Request: A third-party auditor needs read-only access to your production AWS account for 48 hours. They should only be able to view CloudWatch logs, EC2 instances, and billing dashboards. Design the access mechanism.

A. I'd create a dedicated IAM role — not a user — with a scoped read-only policy: CloudWatch Logs read (FilterLogEvents, GetLogEvents, Describe*), ec2:Describe*, and the billing views (aws-portal/ce read actions or the BillingReadOnly managed policy). The trust policy allows the auditor's identity to assume it — ideally federated through our IAM Identity Center as a temporary user assignment, or if they come from their own AWS account, a trust with ExternalId to prevent confused deputy. Enforce MFA in the trust condition and set MaxSessionDuration to a short window. The 48-hour limit is handled with an expiry: either an explicit Condition with aws:CurrentTime DateLessThan the end date in the role policy, or a scheduled pipeline/EventBridge job that deletes the role after 48 hours — I prefer both, belt and suspenders. All their activity is captured in CloudTrail under the assumed-role session name for the audit trail.

---

### 9. Confused Deputy Problem: Your SaaS vendor asks you to create an IAM role with a trust policy that allows their AWS account to assume the role. How do you prevent the confused deputy problem? What condition key is critical here?

A. The confused deputy problem is when the vendor's service — which has permission to assume roles in many customers' accounts — gets tricked into using its access on the wrong customer's behalf; an attacker who knows my role ARN could ask the vendor to assume my role. The critical control is the ExternalId condition. The vendor gives each customer a unique ExternalId, and my role's trust policy requires "sts:ExternalId": "that-value" in the AssumeRole call. An attacker signing up to the same vendor cannot make the vendor present my ExternalId, so the assume fails. I'd also scope the trust to the vendor's specific account and role, grant the minimum permissions the integration needs, and monitor assume events in CloudTrail. Modern variants also use aws:SourceArn/aws:SourceAccount conditions for service-to-service trust.

---

### 10. Access Key Rotation at Scale: Your organization has 300 IAM users with active access keys. Some keys haven’t been rotated in 18 months. Design a strategy to enforce 90-day rotation, identify stale keys, and automate deactivation without breaking production workloads.

A. First, visibility: pull a credential report (aws iam generate-credential-report) to inventory all keys, ages, and last-used timestamps. Anything unused for 90+ days gets deactivated right away after owner confirmation. Then the strategy in phases — Phase 1: classify keys into human vs workload. Humans move to SSO/federation, killing their keys entirely. Workloads move to roles (instance profiles, IRSA, OIDC for CI/CD) wherever possible — rotation problems disappear when there are no long-term keys. Phase 2: for the residual keys that genuinely must exist (legacy on-prem apps), automate rotation: a Lambda on an EventBridge schedule checks key age via the credential report, notifies owners at 75 days, creates the second key slot, coordinates cutover via Secrets Manager, deactivates at 90, deletes at 100. Never hard-delete first — deactivate, wait for screams, then delete. Phase 3: enforce — an AWS Config rule (access-keys-rotated) for compliance dashboards, and alerts on non-compliant keys. The real win is shrinking 300 users with keys down to a handful.


## 1C. Follow-up / Deep Dive Questions:

---

### 1. You mentioned least privilege — how do you actually determine what permissions a service or user needs? What tools does AWS provide for this?

A. I start from evidence, not guesswork. For an existing workload: IAM Access Analyzer can generate a policy directly from the role's CloudTrail history — it writes the policy covering only the actions actually used, which I then review and tighten resource ARNs on. IAM Access Advisor (last-accessed data) shows which services a role touched and when, so I can strip services never used. For a new workload, I begin with the narrowest guess from the app's documented AWS calls, deploy to dev, and iterate on AccessDenied errors — in dev, not prod. CloudTrail is the ground truth throughout. I also periodically re-run the unused-access review because permissions rot: apps stop using things but policies never shrink on their own.

---

### 2. You said you’d use IAM roles instead of access keys. But what if the workload is running outside AWS — on a developer laptop or an on-premise Jenkins server?

A. Three good options depending on the case. For on-prem servers like that Jenkins box, IAM Roles Anywhere — you register a certificate authority with AWS, the server authenticates with an X.509 certificate and exchanges it for temporary role credentials; no stored AWS keys. For CI/CD platforms that support it, OIDC federation is even cleaner — GitHub Actions, GitLab, etc. exchange their identity token for role credentials per job. For developer laptops, humans should use SSO: aws sso login through IAM Identity Center gives short-lived credentials in the CLI. The last resort is an IAM user with tightly scoped permissions, MFA, and enforced rotation — but in my experience once Roles Anywhere and OIDC are set up, there's almost no legitimate need for long-term keys anywhere.

---

### 3. You mentioned Permission Boundaries. Can you draw the Venn diagram of how effective permissions are calculated when a user has an identity policy, a permission boundary, and an SCP all applied simultaneously?

A. Conceptually it's three overlapping circles — SCP, permission boundary, identity policy — and the effective permissions are only the region where all three intersect. The SCP defines what the account may do at all; the boundary defines the ceiling for that specific identity; the identity policy defines what was actually granted. Remove any one circle's coverage of an action and the action is denied: SCP allows it + boundary allows it + identity policy doesn't grant it → denied; identity policy grants it + boundary allows it + SCP doesn't → denied. And an explicit Deny in any of them cuts through the intersection unconditionally. Resource-based policies sit slightly outside this intersection model — within the same account they can independently allow a principal — but SCP and explicit denies still always apply.

---

### 4. If SCP cannot grant permissions, only restrict them — where does the actual permission grant come from in a multi-account setup?

A. The grant always comes from policies inside the member account — an identity-based policy attached to the user or role, or a resource-based policy on the resource being accessed. SCPs are pure filters sitting above. So in a multi-account setup, the typical flow is: IAM Identity Center provisions permission sets, which materialize as actual IAM roles with real allow policies inside each member account — that's the grant. Or for workloads, Terraform creates the execution roles with their policies in each account. The SCP then bounds what those grants can express. No SCP, no boundary, no trust policy grants anything by itself; something in the account must say Allow.

---

### 5. You said you’d use aws:PrincipalOrgID in the bucket policy. What happens if the requesting principal is a service-linked role? Does this condition key still work?

A. Good subtle case. aws:PrincipalOrgID matches the organization of the principal making the request. A service-linked role lives in my account, and my account is in my organization — so requests made by that SLR still carry my org ID and the condition works fine. The place it breaks is different: when an AWS service principal itself (like cloudtrail.amazonaws.com delivering logs) makes the call directly rather than through a role in my account, there's no organization membership at all, so a bucket policy that denies everything without my PrincipalOrgID would block the service delivery. That's why those deny statements need an exception — the aws:PrincipalIsAWSService context key, or explicit allow statements for the service principals with aws:SourceAccount/SourceArn conditions. I hit exactly this with a central CloudTrail bucket.

---

### 6. How is iam:PassRole different from sts:AssumeRole? Why is PassRole considered a dangerous permission?

A. sts:AssumeRole is about becoming a role — you exchange your identity for the role's temporary credentials and act as it. iam:PassRole is about handing a role to a service — when I launch an EC2 instance with an instance profile or create a Lambda with an execution role, I need iam:PassRole on that role; the service then uses it, not me. PassRole is dangerous because it enables indirect privilege escalation: if I can pass any role and create a Lambda or EC2 instance, I can pass an admin role to a compute resource I control and execute anything with the admin's permissions — without ever being allowed to assume the role myself. That's why PassRole must always be scoped to specific role ARNs and constrained with iam:PassedToService conditions, and why broad iam:PassRole on * is one of the first things I flag in a policy review.

---

### 7. You said IAM policies are JSON documents. What is the difference between Effect: Allow on Resource: * versus Effect: Allow on a specific ARN? How does the wildcard interact with condition keys?

A. Resource: * means the allow applies to every resource the action supports — s3:GetObject on * lets you read any object in any bucket in the account. A specific ARN scopes it to one bucket or key prefix, which is what least privilege demands and what I do in production wherever the service supports resource-level permissions. Condition keys apply on top of whatever resource scope is defined — they don't narrow the resource, they narrow the circumstances: the same s3:GetObject on * combined with a condition on aws:PrincipalOrgID, aws:SourceVpce, or s3:prefix only permits requests matching those context values. So a wildcard-plus-conditions policy can still be reasonably tight, but I prefer explicit ARNs plus conditions: the ARN limits what, the conditions limit when and from where. Also worth noting some actions don't support resource-level scoping at all and legitimately require * — those I isolate in their own statements.

---

### 8. Walk me through what happens at the API level when an EC2 instance with an instance profile makes an API call. How does the instance get the temporary credentials?

A. The instance profile is the container that attaches the role to EC2. When the instance needs credentials, the SDK/CLI queries the Instance Metadata Service — with IMDSv2 it first does a PUT to get a session token, then calls `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>` with that token. Behind the scenes, EC2 has already performed an AssumeRole on the instance's behalf and caches the resulting temporary credentials — access key, secret key, session token — in the metadata service, refreshing them automatically before expiry (roughly every few hours). The SDK credential chain picks them up transparently, so the application code never handles keys. Security notes I always apply: enforce IMDSv2 with a hop limit of 1 to stop SSRF-style credential theft, and remember every API call made this way appears in CloudTrail as the assumed role session tied to the instance ID.


---

### 9. You mentioned ABAC (Attribute-Based Access Control). How does it differ from RBAC and when would you prefer one over the other in a large enterprise?

A. RBAC grants through roles mapped to job functions — you define a Developers role with a fixed policy, and membership determines access. It's simple but explodes at scale: new team, new project, new environment each need new roles and policy updates. ABAC grants through tags/attributes: one policy says a principal can act on resources whose tag matches the principal's tag — for example, allow ec2 actions where ec2:ResourceTag/team equals aws:PrincipalTag/team. Onboard a new team and nothing changes in IAM — you just tag their identities and resources consistently. In a large enterprise I'd prefer ABAC for workload/resource access where tagging discipline exists (it lives or dies on tag governance — enforce tags via SCPs and IaC), and keep RBAC for coarse persona-level access via Identity Center permission sets. In practice we ran a hybrid: RBAC personas for humans, ABAC conditions for team-scoped resource control.

---

### 10. If you delete an IAM role that is currently being assumed by a running Lambda function, what happens to the function’s in-flight executions?

A. In-flight executions keep running — Lambda already holds the temporary credentials for the current execution environment, and STS credentials remain valid until they expire even if the role behind them is deleted; deleting a role doesn't revoke outstanding sessions. The failures show up afterwards: new invocations that require fresh credential assumption fail with an authorization error because the execution role no longer exists. One more subtlety — if you recreate a role with the same name, it's a different principal internally (different unique ID), so anything referencing the old role's unique ID (like certain trust or resource policies that resolved the old ID) may still misbehave. If the goal was revoking a compromised role's active sessions, deletion isn't the tool — you attach a deny-all policy or use the revoke-sessions option that denies tokens issued before a timestamp.

## Architecture Thinking Questions:

---

### 1. Design an IAM strategy for a startup that currently has 5 engineers but plans to scale to 50 within a year. Consider AWS Organizations, SSO, role design, and policy management. How do you build this to scale?

A. At 5 engineers, keep it simple but build the bones that scale. Day one: AWS Organizations with management, production, staging/dev, and a sandbox account — accounts are the strongest isolation boundary and adding them later is painful. IAM Identity Center from the start, even with the built-in directory — no IAM users, ever; migrate to Google Workspace/Okta as IdP when ready. Define three or four permission sets (Admin, Developer, ReadOnly, Billing) rather than per-person policies. All workload roles and policies in Terraform from day one so IAM changes are code-reviewed. Baseline SCPs: region restriction, deny CloudTrail tampering, deny root access keys. Org-wide CloudTrail to a logging account. As they grow to 50: add OUs per environment, split teams into IdP groups mapped to permission sets, introduce permission boundaries for self-service role creation, and add Access Analyzer reviews. The startup mistake to avoid is everyone-is-admin-in-one-account — retrofitting that at 50 people is a migration project; doing it right at 5 costs a week.

---

### 2. You are architecting a multi-account strategy with separate accounts for Dev, Staging, Production, Security, Logging, and Shared Services. Design the IAM role trust relationships and SCP guardrails for cross-account access.

A. Structure: Security and Logging accounts are the sensitive core; Shared Services hosts CI/CD and tooling; workload accounts are Dev, Staging, Prod. Trust design — humans: Identity Center in the management account provisions permission set roles into every account; no direct cross-account trust between workload accounts for humans. CI/CD: the pipeline in Shared Services assumes a dedicated deployer role in each workload account (trust scoped to the specific pipeline role ARN, with conditions); Prod's deployer role additionally requires the pipeline context — no human can assume it. Security account holds an audit/read-only role and an incident-response role trusted into all accounts, assumable only by the security team with MFA. Logging account's buckets accept delivery from all accounts via resource policies with aws:PrincipalOrgID/SourceAccount — nothing assumes into logging. SCP guardrails per OU: workload OU denies leaving approved regions, denies CloudTrail/Config modification, denies deleting the logging role; prod OU additionally restricts iam:* to the IaC deployer with boundaries; sandbox OU is looser but budget-capped. Principle throughout: trust flows from center outward, is one-directional, scoped to named ARNs, never account-root-to-account-root wildcards.

---

### 3. How would you implement just-in-time (JIT) privileged access for production accounts? An engineer should be able to request elevated access for 1 hour with full audit trails and approval workflows.

A. I'd build it on temporary elevated role assumption with an approval gate. Baseline: engineers hold read-only in prod. Elevation flow: engineer requests via Slack workflow or an internal portal with a reason and ticket reference → approval by on-call lead (auto-approve for some tiers, human approval for admin) → automation (Step Functions or the identity platform) grants a time-boxed assignment. Mechanically, the cleanest implementation is IAM Identity Center's APIs to add the user to an elevated permission-set assignment and remove it after 60 minutes via a scheduled job; the alternative is a break-glass role whose trust policy the automation temporarily opens for that user, always with MFA required and a one-hour max session duration. Audit: every session is CloudTrailed under the user's session name; the request, approval, grant, and revocation land in an immutable log tied to the ticket; a recording of session activity (CloudTrail Lake query per session) attaches to the ticket automatically. AWS now offers this natively as Identity Center's temporary elevated access / just-in-time features, and third-party options exist, but I'd keep the approval workflow in our own tooling for flexibility.

---

### 4. Your company is migrating from on-premise Active Directory to AWS. 2,000 users need SSO access to 20 AWS accounts with different permission levels. Design the federation architecture. What are the tradeoffs between AWS IAM Identity Center (SSO) and custom SAML federation?

A. I'd federate rather than migrate users into IAM. Architecture: keep AD as the source of truth, sync identities to the cloud IdP layer, and use IAM Identity Center connected to it — either via AD Connector / AWS Managed Microsoft AD directly, or through Azure AD/Entra as the SAML+SCIM IdP if the company is already moving there. AD security groups map to Identity Center groups (SCIM-provisioned), which map to permission sets across the 20 accounts — so access control stays a group-membership operation in AD, which the helpdesk already knows how to do. Identity Center vs custom SAML federation: Identity Center gives centralized assignment management across all accounts, the user portal, CLI SSO with short-lived credentials, SCIM provisioning, and no per-account IAM identity-provider plumbing — custom SAML requires configuring providers and roles in each of the 20 accounts and users juggling role ARNs, but offers more control for exotic requirements (unusual assertion mapping, multi-IdP setups, per-account customization). For 2,000 users and 20 accounts, Identity Center is the right call unless there's a hard requirement it can't meet; custom SAML federations at that scale become an undocumented mess of role trust policies.

---

### 5. Design an IAM architecture where application teams can create their own IAM roles for Lambda and ECS tasks, but cannot escalate privileges beyond what is allowed. What combination of Permission Boundaries, SCPs, and tag-based policies would you use?

A. Three layers working together. Layer 1 — permission boundary: publish a standard boundary policy (allow the common workload services, deny iam:*, organizations:*, account-level actions) and require it on every role application teams create. Layer 2 — SCP enforcement: an SCP that denies iam:CreateRole and iam:AttachRolePolicy unless the request includes our boundary (Condition: iam:PermissionsBoundary equals the boundary ARN), and denies iam:DeleteRolePermissionsBoundary and modifications to the boundary policy itself except by the platform team role. That makes the boundary impossible to omit or strip. Layer 3 — tag/path-based scoping: teams may only create and manage roles under their own path or team tag (iam:* actions conditioned on aws:RequestTag/team matching aws:PrincipalTag/team), and iam:PassRole is scoped so their compute can only receive their own roles, conditioned on iam:PassedToService. Result: full self-service inside the sandbox — a team can attach any policy they like to their roles, but effective permissions never exceed the boundary, they can't touch other teams' roles, and they can't escalate by passing privileged roles to compute they control. IaC modules with the boundary pre-wired make the golden path also the easy path.

## Practical Troubleshooting Questions: 

---

### 1. A production EC2 instance suddenly cannot write to an S3 bucket. It was working yesterday. No one changed the IAM role or policy. What are all the possible causes?

A. "No one changed anything" usually means something changed elsewhere. My checklist: (1) Bucket side — was the bucket policy changed or an explicit deny added by another team, did Block Public Access or an ownership setting change, did the object prefix being written to change? (2) Encryption — did the bucket switch to SSE-KMS or the KMS key policy change? The role now needs kms:GenerateDataKey; this is the most common cause in my experience. (3) Org level — was a new SCP attached or a permission boundary applied to the role? (4) Network path — if it writes through an S3 VPC endpoint, was an endpoint policy added or the route table changed? DNS or endpoint changes can also surface as timeouts rather than denials, so the exact error matters. (5) Credential side — is the instance actually still getting role credentials (IMDS reachable, IMDSv2 hop limit after containerization)? (6) Different failure than assumed — token expired, clock skew, or the app deployed with baked-in stale keys. CloudTrail's error message for the failed PutObject narrows it to the exact policy type in one look.

---

### 2. An IAM user reports “Encoded authorization failure message” when trying to launch an EC2 instance. How do you decode this message and what information does it reveal?

A. That message appears when the denial details might contain privileged information — AWS encrypts the explanation. You decode it with: `aws sts decode-authorization-message --encoded-message <blob>`, which requires the sts:DecodeAuthorizationMessage permission. The decoded JSON reveals exactly what you need: the principal that made the call, the action and resource evaluated, and crucially which policy type caused the denial — whether an explicit deny matched or no policy allowed it, including SCP or boundary involvement, plus the context values (source IP, VPC endpoint, tags) at evaluation time. For an EC2 launch specifically, the culprit is very often missing iam:PassRole for the instance profile role, or an SCP condition on instance types, regions, or required tags — the decoded message states it directly instead of leaving you guessing across five policy layers.



---

### 3. Your CloudTrail logs show AccessDenied for an s3:PutObject call, but the IAM policy clearly allows s3:* on the target bucket. What could cause this?

A. An allow in the identity policy is necessary but not sufficient — something else is denying. Candidates in the order I check them: (1) Bucket policy explicit deny — often a deny-unless-condition pattern: aws:SecureTransport false (app using http), missing required encryption headers (s3:x-amz-server-side-encryption), wrong VPC endpoint (aws:SourceVpce), or IP restrictions. (2) KMS — bucket default encryption with a CMK the principal can't use; the error surfaces as S3 AccessDenied. (3) SCP blocking s3 actions or the region. (4) Permission boundary on the role lacking S3. (5) VPC endpoint policy restricting which buckets/actions pass through. (6) Session policy — if the app runs on an assumed role with a scoped-down session, the session policy caps permissions regardless of the role policy. (7) Cross-account nuance — bucket owned by another account, so the identity policy alone can't grant. The decoded authorization message or CloudTrail data-event detail identifies which layer, so I don't guess.

---

### 4. After enabling an SCP to restrict regions, your existing Lambda functions in allowed regions start failing. The Lambda code hasn’t changed. What went wrong?

A. The SCP almost certainly caught legitimate global/cross-region calls the functions depend on. Common failure modes: the functions call services that only exist in us-east-1 or as global endpoints — STS global endpoint, IAM, CloudFront, Route 53 — and the SCP's allowed-region condition blocks them because the exemption list (NotAction for global services) was incomplete. Another classic: the Lambda execution role calls sts:AssumeRole via the global STS endpoint sts.amazonaws.com, which resolves to us-east-1 — if us-east-1 isn't allowed and STS isn't exempted, every assume fails; fix is exempting STS or configuring regional STS endpoints. Also possible: the functions read a replicated secret/parameter or write logs/metrics to another region, or use S3 buckets homed elsewhere. Diagnosis is straightforward — CloudTrail shows AccessDenied events with the SCP as the failing policy and the region attempted. Fix: extend the SCP's exemptions properly in a sandbox OU first, which is the lesson — SCPs get tested against real workload traffic before org-wide rollout.

---

### 5. A developer created an IAM role using Terraform, but when they try to assume it immediately after creation, they get AccessDenied. Five minutes later, it works fine. Explain this behavior.

A. That's IAM's eventual consistency. IAM is a global service — creates and updates happen in one place and propagate asynchronously to every region's evaluation endpoints. Terraform gets a success response as soon as the control plane accepts the CreateRole, but the role (and its trust policy) hasn't replicated to whereever the AssumeRole is being evaluated yet, so STS rejects it briefly. Five minutes later replication has caught up — usually it's seconds, occasionally longer. Handling it properly: build retry with exponential backoff into anything that assumes or passes a freshly created role (most AWS SDKs and Terraform providers do retry specific InvalidPrincipal/AccessDenied cases for this reason), add explicit dependencies or a short wait in pipelines that create-then-use roles, and in Terraform use create-then-consume patterns in separate applies or rely on the provider's built-in propagation retries rather than sleeps where possible. It's not a bug — it's a documented property of IAM you design around.


# AWS Scenarios

Document AWS DevOps scenarios, troubleshooting cases, and practice examples here.

## 1. what is cache busting?
A. Cache busting is a web development technique that forces browsers to fetch the latest version of a file rather than using an outdated, locally cached copy.

If you update your website's code, returning visitors might still see the old version because their browser believes it already has the correct file. By changing the file's URL—such as modifying style.css to style.css?v=2 or style.a1b2c3.css—the browser treats it as a completely new resource and bypasses the cache to download the updated file from the server.

**Common Cache Busting Methods:**

**File Hashing (Recommended):** Build tools (like Webpack, Vite, or Gulp) automatically rename files based on their content (e.g., app.892jf9.js). Whenever you change the code, the hash changes, ensuring the browser fetches the new file.

**Query Strings:** Adding parameters like ?v=1.2 or a timestamp to the end of your file's URL in your HTML. While easy to implement, some proxies and CDNs struggle to cache files with query strings effectively.

**File Path Versioning:** Changing the directory structure (e.g., /v1/style.css to /v2/style.css) to signal an update.