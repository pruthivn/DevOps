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

github actions : CI/CD platform build into github. Track changes happening on our repo 24/7. Whenever something happens (push, pr) it automatically handles the tasks we configure.


--> Native github integration
--> YAML configuration
--> marketplace with pre-build actions
--> Multi-os : We can run/test apps across diff OS.
--> matrix build : Test across multiple version
--> 2000 Mnts/Month free for Private Repos.. Unlimited for public repos
--> No addtional dedicated tools required for Secrets management.

===

Core components of Github Actions : 

---

1. Workflow : A complete automation of our process defined in a YAML file. 
This file should be in "Repo --> .github --> workflows --> name.yml"

---

2. Event : An event is the trigger that starts our workflow.

Common event types : 
push 				: When any code is pushed to repo.
pull_request		: When a PR is created
schedule			: run at a specific time (cron)
workflow_dispatch	: Manually trigger from Github

---

3. Job : A group of steps that run on a machine (runner)

1. Verifying code quality
2. Preparing a build
3. result print

---

4. Step : A single task inside a job.

---

Runner : This is a virtual machine that executes our jobs.

Github hosted runners : ubuntu, windows, macos (Managed by GitHub)
Self-hosted runners : your ec2 instance or other manages server can be a runner.

---

## Different ways of triggering pipeline

1. on push: when you push the code to github pipeline will trigger.

```yaml
name: myfirst workflow
on: push
jobs:
```
2. when we rise PR then trigger pipeline

```yaml
name: PR workflow
on: 
  pull_request:
    branches:
      - main
jobs:
```
3. if we workflow_dispatch we manually need to trigger the pipeline in github console

```yaml
name: manual dispatch
on:                          or      # on: workflow_dispatch this also vaild
  workflow_dispatch:        
jobs:
```
4. for specific branch: pipeline will trigger only when we push the code to feature or dev branch.

```yaml
name: Push on specific branch
on: 
  push:
    branches:
      - feature
      - dev

```
5. Scheduled basis: below pipeline will trigger every mon-fri 9 am.

```yaml
name: Scheduled workflow
on: 
  schedule:
    - cron: '0 9 * * mon-fri'
```

6. Pipeline will trigger for multiple events

```yaml
name: Multiple events
on: 
    push: 
        branches: [main, demo]
    pull_request:
        branches: [main]
    workflow_dispatch:
```
## Conditional statements
In the below code if head refers/points to main then only run the job.

```yaml
jobs:
  only_on_main:
    if: github.ref == 'refs/heads/main'
    name: Only on main branch
    runs-on: ubuntu-latest
    steps:
      - name: shell script
        run: echo "running on main branch"
```
## dependency stages
we use needs key for in the below code if code stages pass then only build stage executes.
```yaml
jobs:
  code:
    runs-on: ubuntu-latest
    steps:
      - name: clone the code from github repo
        run: echo "Code cloned from the github repo"
        
  build:
    needs: [code]
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        run: echo "Docker image build succesfully"
```

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

## matrix in Githubactions
A matrix is a configuration strategy that lets you run a single job multiple times using different variables.
in the below example we are testing our nodejs app with differnet versions in different environments

```yaml
jobs:
  matrix-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [12, 14, 16]
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node.js ${{ matrix.node }}
        uses: actions/setup-node@v6
        with:
          node-version: ${{ matrix.node }}
      - run: npm install
      - run: npm test.js
```

## Variables

workflow variables : $NAME, $VAR_NAME
we can define workflow varibales like above for varables stored in github we use below format.
Config Varibales : ${{ vars.VAR_NAME }}

**Note:** varaible precedence in workflow level:  steplevel > joblevel > workflowlevel

Overall vars precedence: Step Level > Job Level > Workflow Level > Environment Level > Repository Level > Organization Level

```yaml
name: Workflow level variables

on: workflow_dispatch

env:
    VAR: workflow level variable
# steplevel > joblevel > workflowlevel
#Picks from workflow level variable    
jobs:
    print-cloud:
        runs-on: ubuntu-latest
        steps:
            - name: Print cloud variable
              run: echo "I am running on $VAR"
#Job level variable overrides workflow level variable
    print-cloud-again:
        runs-on: ubuntu-latest
        env:
            VAR: job level variable
        steps:
            - name: Print cloud variable again
              run: echo "I am running on $VAR"
#Step level variable overrides job and workflow level variable
    print-cloud-step-level:
        runs-on: ubuntu-latest
        steps:
            - name: Print cloud variable again
              env:
                VAR: step level variable
              run: echo "I am running on $VAR"
#config variables are defined at the repository level and can be used across all workflows in the repository. They are accessed using the vars context and take precedence over workflow, job, and step level variables.
    print-repo-level-variable:
        runs-on: ubuntu-latest
        steps:
            - name: Print project variable 
              env:
                VAR: repository level variable
              run: echo "I am running a project named ${{ vars.VAR }}"
```

context variables : In GitHub Actions, contexts are objects that store information about your workflow runs, runner environments, jobs, and steps.

ref:https://docs.github.com/en/actions/reference/workflows-and-actions/contexts

 ```yaml
 name: context variables example

on: 
    push:
        branches:
        - "*"
jobs:
    print-cloud:
        runs-on: ubuntu-latest
        steps:
            - name: Print repo name
              run: echo "I am on this repo ${{ github.repository }}"
            - name: Print workflow name
              run: echo "I am running this workflow ${{ github.workflow }}"
            - name: Triggered by
              run: echo "Triggered by ${{ github.actor }}"
```

for secrets we call secrests in this way "{{ secrets.access_key }} we con't see secrets in logs.

```yaml
  - name: Using to test the secret variable
    run: echo "THis access key is used to access AWS environment ${{ secrets.AWS_ACCESS_KEY }}"
  - name: Using to test the secret variable-2
    run: echo "THis secret key is used to access AWS environment ${{ secrets.AWS_SECRET_KEY}}"
```
=====

# OIDC provider for connecting AWS to GithubActions

TO access AWS environment from GHA using OIDC method (Secured)

Step 1 : create OIDC provider with below info. 
Navigate to IAM in AWS click on identity providers click on add provider and add the below values in provider url and auidence section. Make sure no Blank spaces on provider url section.

provider: token.actions.githubusercontent.com
audience: sts.amazonaws.com
![alt text](.images/image.png)

Step 2 : Create an IAM role for GHA.

Click on create role and then choose "web identity"

provider: token.actions.githubusercontent.com
audience: sts.amazonaws.com
GitHub organization : <provide github username>
![alt text](.images/image-1.png)


Step 3 : Add required polcies to access resources in AWS from yaml file.

Step 4:  Create and grab the role arn

role : arn:aws:iam::655700896650:role/GHA-Role

```yaml
name: AWS OIDC example

on:
  push:
    branches: main
permissions:
  id-token: write  # Allows the workflow to request an OIDC token from GitHub.
  contents: read   # Allows the workflow to read repository files.
jobs:
  aws-access:
    runs-on: ubuntu-latest
    steps:
      - name: checkout the code
        uses: actions/checkout@v6
      - name: configure aws credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v6.1.0
        with:
          aws-region: us-east-2
          role-to-assume: arn:aws:iam::655700896650:role/GHA-Role
      - name: verify the user
        run: aws sts get-caller-identity
      - name: list s3 buckets
        run: aws s3 ls
```
**Note:** while creating identity privider we are telling aws to please save GitHub's address (https://githubusercontent.com) and public keys(aws know about keys using OIDC Discovery) in your contact list so you can trust them later.

![alt text](.images/image-2.png)

## Runners
### github hosted runner : 
GitHub gives a VM --> THis VM runs our Job --> Destroys the VM
--> max a job can run for 6 hours

### Self-Hosted Runner : 
We can setup our own machine --> Github sends jobs to it --> Machiebe stays alive.
--> Can access private VPC / Databses
--> We can have pre-installed tools (docker, kubectl)
--> We can use IAM roles without storing any credentials at GHA.
--> We can consider for jobs that runs more than 6 hrs period.


sudo dnf update -y
while configuring the ec2 instance into a selfhosted runner it will ask some dependences we need to install those.

dnf install dotnet-sdk-8.0 libicu -y


./run.sh

sudo ./run.sh install
sudo ./run.sh start

```yaml
name: Create S3 Buckets

on:
  workflow_dispatch:
  
jobs:
  s3-list:
    runs-on: self-hosted # it will pick randomly
    steps:
      - name: list S3 bucket
        run: aws s3 ls
  
  s3-create:
    runs-on: [self-hosted, dev] # for running in a specific instance
    steps:
      - name: list S3 bucket
        run: aws s3 ls
```
if you want run a workflow in specific self-hosted runner(instance) while configuring runners realted things in ec2 instance while performing below command we need to add --lables option
```sh
./config.cmd --url https://github.com/pruthivn/GHA_practice --token AYCSRJIHCIIGC4BGUVWZFPLKIYYDK --label dev
```
then we need to mention that in yaml file see the above yaml


# reusable modules/workflows

we use workflow_call in module and use "uses" in actual/main yaml file like below.
main.yml
```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  workflow_dispatch: 

jobs:
  code-quality:
    uses: ./.github/workflows/code-quality.yml

  dependency-scan:
    uses: ./.github/workflows/dependency-scan.yml
```
dependency.yml
```yaml
name: Dependency scan

on:
  workflow_call:

jobs:
  validate:
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false # if we put true if one matrix job fails rest of the matrix jobs didn't execute.
      matrix:
        python-version: [3.12]
        
    steps:
      - name: checkout code
        uses: actions/checkout@v6

      - name: Setup python 3.12
        uses: actions/setup-python@v6
        with:
          python-version: 3.12
          
      - name: Install pip-audit
        run: pip install pip-audit

      - name: Audit Dependencuies for known CVE
        run: pip-audit -r requirements.txt
```