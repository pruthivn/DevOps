# AWS

Snapshot : backup copy of an EBS volume. 

**Note:**EBS volumes are zone specific(we can't copy/attach ebs volumes from one zone to other zone if we want to attach we will create snapshots and create volume from that snapshot in desired zone) and ebs snapshots are region specific.

for region to region we will copy snapshot to desired region from that snapshot we will create volume.

we can cpy snapshots form one account to another account. we can share snapshots everyone publicly as well using snapshot id.

Snapshots are point in time copies. (If you create a snapshot at 09:30, whatever the files you have in volume at 09:30, only those will be backedup).. 

AWS uses s3 as backend platform for snapshots.

--> Can we see what data we have inside the snapshot.?Ans; no..
--> AWS uses incremental backup mechanism (in backend).. We just need most recent copy to restore.. multiple copies not required..
--> If a volume is encrypted, and if we create a snapshot, snapshot also enabled with encryption defaultly.

--> If any snapshot if encrypted with "default encryption key", we cannot share it with other aws accounts.
--> If any snapshot if encrypted with "custom encryption key", we can share it with other aws accounts.. but we need to share the encryption key also.


------------


### DLM: Data Lifycle Manager:
Designed to automate the snapshot creation. Based on environment or org/proj requirement, we configure this. (RTO / RPO)

1. RTO (Recovery Time Objective)
Simple Meaning: Your downtime limit.
The Question: "How long can we afford to be offline?".
Focus: Recovery speed.
Example: If your RTO is 2 hours, you must have your systems back up and running within 2 hours of a crash to avoid serious business damage. 

2. RPO (Recovery Point Objective)
Simple Meaning: Your data loss limit.
The Question: "How much data can we afford to lose?".
Focus: Backup frequency.
Example: If your RPO is 1 hour, you must back up your data at least every hour. If the system crashes, you might lose the last 59 minutes of work, but you won't lose more than an hour

### Creating DLM in AWS

**Step-1:** Goto DLM or on EC2 console on leftpane under ebs store click on lifecycle manager select default or custom policy then in target resource tab it will show 2 options instance(creates snapshots for all the volumes attached to this instance) or volumes then we need to provide the tags of the instance/volume according to this we will select instance or volume. automatically a role, *policystatus --> select enabled*(if disable it will create snapshots till you enable it) then*Excludevolumes --> select if you want exclude volumes(root volume(/) or other volumes)* we are creatin volumes according to tags if we want igonore some volumes with we can mention them.
![alt text](.images/dlm1.png)
![alt text](.images/dlm2.png)
![alt text](.images/dlm3.png)

**Step-2:**in schedue details tab we schedue how frequently we need to create snapshots(like everyday or hours or weeks) *rentenion type* here is how many days or how many latest snapshots we can retain it has 2 options 
1. count -- last 5 or 10(or number we gave) latest snapshots we can specify number
2. Age -- if we select age we can  give how many weeks or days snapshots we can retian(if we give 2 last 2 weeks snapshots reatined) if it cross 1 week that snapshot automatically delted by AWS.

**Note:** we can add multiple schedules.

**Advanced Setting:**
1. *Tagging* we can give tags to created snapshots
2. *pre and post scripts* -- if you want to run any scripts in instance before creating snapshot.
3. *Fast snapshot restore*
4. *Cross-region* if we want to copy our snapshots to across the regions, we can set expire date for that s copied snapshot in another region(we copied snapshot to us-east-1 we set expire 2 days after 2 days it will automatically deleted).
5. *Cross-Account* if we want to share created snapshots to different accounts.


## Golden AMI:
**OS Hardening:** OS hardening is the process of securing a computer system by reducing its attack surface, closing security gaps, and removing unnecessary features. The main goal is to make the system safe from hackers and malicious attacks.

The *Center for Internet Security (CIS)* acts as the global standard setter for OS hardening by providing the specific blueprints, tools, and pre-secured systems needed to protect your computer. Instead of guessing which settings to change, IT professionals use CIS resources to safely lock down their operating systems

CIS benchmark list: https://www.cisecurity.org/cis-benchmarks

Base Instance --> httpd, custom web content, user add, password auth enable, tree, 2gb addl volume... --> Checklist --> Reboot and verify --> Sanity check --> Then create a GoldenAMI

**Note:** when we create a Golden AMI form a base instance in the backend AWS will create snapshots of a ebs volumes attached base instance and associates those sanpshots to Golden AMI when we create an instance with this golden AMI those snapshots will be used and instance will launched.

### Creating Golden AMI:
**Step-1:** select the *instance > actions > Images & templates > CreateImage* fill the deatils if we select the *reboot intance* if we are creating the ami on a running instance it will reboot that instance so data will be at rest for data consistency. then we ebs volume setttings.
![alt text](.images/Goldenami.png)
![alt text](.images/Goldenami2.png)

we can't delete the snapshots attached to ami's.

We have *EC2 image builder* to automate the above golden image creation process.EC2 Image Builder is a fully managed AWS service that automates the creation, management, and deployment of customized, secure, and up-to-date virtual machine and container images. 


13.200.251.170
13.201.13.166		--> Current IP

EIP : Elastic IP Address : We can allocate a dedicated IP Address to instance. 
In general, if we perform stop and start operqation Instance auto allocated public ip will change.



We always enbale these 2 options while launching instance to protect our resources from accidental deletion/stop operations on instance. we can also enable this from *Actions* tab as well 
Stop Protection : 
Termination Protection : 

Shutdown behaviour : What should happen to the ec2 instance when we initialise the shutdown at OS level. Default option is "Stop"..  


## Elastic file system(EFS):
EFS uses NFS, NFS port number is 2049. NFS uses 4.1 protocol.

NFS Supports linux OS only, THat means EFS supports only linux Instances.
For windows, We have another service "FSx", FSx works with SMB (Server Message Block) Protocol.

EFS has unlimited storage.
No Pre-provisioning required


**Step 1 :** Create a Security group for your ec2 instance

sg-01f2aff5e6ac57dc1 - Web-SG		(Opened port http/80 and ssh/22)


**Step 2:**  then create a SG for EFS. Port : 2049.. Always prefer Option 4

Where we need to open this port.?? 
Option 1 : Open for everyone (least Secured)		: 0.0.0.0/0
Option 2 : Open for 2 ec2 instance Private IPs		: web-1 pvt ip & web-2 pvt ip
Option 3 : Open for entire VPC network				: 172.31.0.0/16

Option 4(Recommended) : Open for EC2 instance Security group. (More Secured and AWS suggested best option) This is also called security group pipeline method.
Open port 2049 and set destination as "Web-SG" (Pipeline Mechanism).. (Data will be encrypted automatically)
 
Option 5 : Attach "Default SG" for your EFS and EC2 instances.. Both can communicate.

--------

#!/bin/bash
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

DocumentRootPath: /var/www/html/

1. Creating EFS: Go to EFS console --> Create file system --> Click on customize --> give name --> if you want oyour EFS to across the region select the region or only AZ select zonea and we have different options explore.
![alt text](.images/EFS.png)

2. Remove the default SG and Add the SG we created for EFS. (Port 2049 opened) and click on create file system.
![alt text](.images/EFS2.png)

3. then we can add policies to our EFS. (like encryption, preventing accesss etc..)
![alt text](.images/EFS3.png)
4. Review all the details and click on create file system. It will take 1-2 mins to create EFS.
![alt text](.images/EFS4.png)
----
**step-3:** after creating EFS, we need to mount it to our ec2 instance. in EFS console click EFS file system and click on attach it will show 2 options mount with ip(it is limited to One AZ only) and DNS(recommended) We can mount it using NFS protocol.
![alt text](.images/EFS5.png)


sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0d0e78183133de983.efs.ap-south-1.amazonaws.com:/ /mount-point

sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0d0e78183133de983.efs.ap-south-1.amazonaws.com:/ /var/www/html/


ec2 : NFS..??? SSH..?? HTTP..??
EFS : 2049.. 

Scaling : 

Vertical Scaling : Adding more resources/components (CPU/memory/ENI) to existing instance.. 

t3.micro		--> Not working as expected --> Cloudwatch --> t3.micro --> t3.small

--> We need to stop the ec2 instance to perform vertical scaling.


--> Change Request / CRQ --> CAB meeting (Change Advisory Board) (Prod --> 1 week / Non-prod --> 2 week)

Dev / QA / SIT --> We can do it (1 week observe)
UAT --> CAB --> 
PRD --> CAB --> UAT CRQ Number --> prod Approve (2 weeks gap)

e-crq --> Emergency crq 

Is this implemnted in lower env.?
Cost change
Downtime required
Client informed
change window / When you planning perform
Pre-Implementation
Actual implementation
Post-Implementation
Validations / Sanity Check
Rollback plan

Tasks list : db, support, db, aws..


## AWS Cloudwatch

Cloudwatch : Monitoring service.. Observability (o11y).. 
This is enabled for most of the aws services we use.. 


we have 2 types of monitorings.

1. Basic Monitoring : 5 min (by default)
2. Detailed monitoring : 1 Min(we need to enable this in monitoring section it will give metrics every 1 min.. it will cost us extra money..)

EC2 : We can monitor CPU, Disk and Network metrics.. We cannot monitor "memory/RAM" usage using default cloudwatch metrics.. 

We can install "CW Agent" inside ec2 instance and we can monitor memory and disk free space.. 

--> We can configure alarm based on the metrics and we can invoke services i.e; SNS (simple notification service) or we can take automated actions on the resources.. 

CPU >=90 for 300 seconds --> reboot
CPU <=20 for 300 seconds --> stop

To monitor memory usage or storage (free storage) or to get our application logs from ec2 to aws.. We need to install Cloudwatch agent.. 

**DL(Distribution List)**: it is a saved group of email addresses that can be used to send emails to multiple recipients at once.
in organisation we don't use specific email address but we have DL (Distribution list) we add this DL mail in SNS topic it will sent alarams and all the team members they will see that mail message.

### Cloudwatch Agent Installation:
To monitor memory usage or storage (free storage) or to get our application logs from ec2 to aws.. We need to install *Cloudwatch agent*. to install this agent we need SSM agent this SSM agent will install in cloudwatch agent in instance.

1. Create a role and required permissions(like CloudWatchAgentServerPolicy, ec2 policies etc.) attach that role to ec2 instance(select instance click on Actions --> Security --> Modify IAM role --> select the role we created and click on update IAM role) and attach the below policies to that role. wait sometime for role to take effect or you can reboot(better reboot to install ssm agent).

2. select ec2 instance goto mointoring tab clickon Configure Cloudwatch Agent if you SSM agent not installed reboot the instance.
![alt text](.images/CWAgent.png).
3. then click install agent and then click on configure Cloudwatch agent you will lot options like memory, disk, cpu, network etc.. select the metrics you want to monitor and also we integrations we can get metrics from prometheus as well. if we select integration like JVM we can java application logs as well(in ur machine app logs are stored in /var/logs/app_name if we eable loggging and give that in integration tab it will those logs in cloudwatch)
![alt text](.images/CWAgent1.png)

4. then click on next and review details and click on Deploy. then you monitor metrics in cloudwatch console or in instance monitoring tab.

**Note:** if we want to install cloudwatch agent in multiple instances we can use system manager to automate the process. we can create a document in system manager and we can run that document in multiple instances to install cloudwatch agent.

============

