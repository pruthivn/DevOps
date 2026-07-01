# GitHub Actions

This directory contains GitHub Actions-related DevOps interview questions and resources.

SDLC : Software development life cycle.

Waterfall methodology : 10 pages website --> We take req for 10 pages, develope 10 pages and deliver 10 pages..
Requirement --> Design --> Implementation --> Verification --> Maintenance

---

Agile methodology : 10 pages website --> We take 1 page, develope 1 pages and deliver 1 page for sprint..
Requirements --> plan --> Design --> Develop --> Release --> Track & Monitor 

Sprint Planning : Spring goal, Sprint plan, Product backlog items.. 
Daily Scrum : Daily 15 min event/meeting.. Progress towards the sprint goal.. Actionalble plan for next day..
Sprint Review : Inspect the outcome, Plan for next sprint, Plan next activities on Product backlogs..
Sprint Retrospective : Review backlog items, PLan to increase quality and efficiency

---

DevOps methodology : 
Slow release cycle
Eliminated manual build / errors
Fixed poor communication between dev team and ops team
Early issue detection

---

CICD Tools 

CI : COntineous Integration : 

CD : Contineous Delivery   : After CI, There is a manaul step added to perform the deployment
	 Contineous Deployment : After CI, Deployment will happen automatically. No manual step involved.
	 
======

github actions : CI/CD platform build into github. Track changes happening on our repo 24/7. Wheever something happens (push, pr) it automatically handles the tasks we configure.


--> Native github integration
--> YAML configuration
--> marketplace with pre-build actions
--> Multi-os : We can run/test apps across diff OS.
--> matrix build : Test across multiple version
--> 2000 Mnts/Month for Private Repos.. Unlimited for public repos
--> No addl dedicated tools required for Secrets management.

===

Core components of Github Actions : 

---

Workflow : A complete autimation of our process defined in a YAML file. 
This file should be in "Repo --> .github --> workflows --> name.yml"

---

Event : An event is the trigger that starts our workflow.

Common event types : 
push 				: When any code is pushed to repo.
pull_request		: When a PR is created
schedule			: run at a specific time (cron)
workflow_dispatch	: Manually trigger from Github

---

Job : A group of steps that run on a machine (runner)

1. Verifying code quality
2. Preparing a build
3. result print

---

Step : A single task inside a job.

---

Runner : This is a virtual machiene that executes our jobs.

Github hosted runners : ubuntu, windows, macos (Managed by GitHub)
Self-hosted runners : your ec2 instance or other manages server can be a runner.

---

Actions : Its a reusable piece of code that does a specific task. Using a specialised tool multiple times in our workflow.

actions/checkout@v4	--> checks out our repo code.
actions/setup-node@v6 --> Setup nodejs

=======


- uses: actions/setup-node@v6

- uses: actions/checkout@v6

actions : official namespace provided by github
checkout : This will fetches code in the workflow to access.
@v6



- name: checkour repo
  uses: actions/checkout@v6
- name: run build
  run : npm build && npm run build
  
========

workflow variables : $NAME, $VAR_NAME
Config Varibales : ${{ vars.VAR_NAME }}
context variables : 
https://docs.github.com/en/actions/reference/workflows-and-actions/contexts

=====

TO access AWS environment from GHA using OIDC method (Secured)

Step 1 : create OIDC provider with below info. 

provider: token.actions.githubusercontent.com
audience: sts.amazonaws.com

Step 2 : Create an IAM role for GHA.

Click on create role and then choose "web identity"

provider: token.actions.githubusercontent.com
audience: sts.amazonaws.com
GitHub organization : <provide github username>

Step 3 : Add required polocies to acess resources from GHA

Step 4 P Create and grab the role arn

role : arn:aws:iam::655700896650:role/GHA-Role

=======

github hosted runner : GitHub gives a VM --> THis VM runs our Job --> Destroys the VM
--> max a job can run for 6 hours

Self-Hosted Runner : We can setup our own machine --> Github sends jobs to it --> Machiebe stays alive.
--> Can access private VPC / Databses
--> We can have pre-installed tools (docker, kubectl)
--> We can use IAM roles without storing any credentials at GHA.
--> We can consider for jobs that runs more than 6 hrs period.


sudo dnf update -y

dnf install dotnet-sdk-8.0 libicu -y


./run.sh

sudo ./run.sh install
sudo ./run.sh start
