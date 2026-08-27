

Lambda : Serverless compute service. 



EC2 : OS patching, Instance Scaling/Auto Scaling, long running application, Max executing - unlimited..
Lambda : Fully managed by AWS, pay for the duration you run your code, short, event driven tasks.. 

lambda handler : This is the entry point to the lambda function. 

IAM Role : Defaultly have permisisons to write logs to cloudwatch logs group.. add more polocies to provide more aws servcies access.


REPORT RequestId: 56976192-bb75-4c7d-989f-fae5856e1689	
Duration: 1.73 ms	
Billed Duration: 94 ms			--> $$
Memory Size: 128 MB	
Max Memory Used: 36 MB			--> $$
Init Duration: 91.33 ms	

Limitation : max timeout is 15 min only.. 
Deployment package size: 50 MB (direct upload).. 250 MB unzipped via s3..
Cuncurrent execution : 1000 per region
layers: 5 layers (250 MB)


Invocation Models:

Synchronous Invocation: The caller sends a request for the respose, lambda runs the function and returns the result directly to the caller. If the function fails, the caller is responsible for retrying.. 
--> API gateway, ALB, Function URL

Asynchronous Invocation: The caller sends a request for the respose, lambda returns with with 202 status code, runs the function and returns the result directly to the caller. If the function fails, it sends the event to DLQ (Dead Letter Queue).. it retries 2 times.. 
--> S3, SNS, Eventbridge

Event Source Mapping : Lambda polls the source for new records and invokes the lambda when it finds any new records / batch of records.. 
--> SQS, DynamoDB, MSK, kenesis..

---

lambda execution lifecycle : 

1. INIT Phase - Initialisation phase - Downloads the code, initialise the runtime and run the code..
2. INVOKE Phase - Executes the handler function. This is the active phase where our code business logic runs.
3. SHUTDOWN Phase - After a period of inactivity, AWS destroys the environment and frees the resources.


Cold start : INIT + INVOKE Phase.. First invoke or invoke after long inactive gap.. We get high latency..

Warm start :  INVOKE Phase only.. Subsequent invocations when env is alive.. Lower latency.. 

How to reduce cold start:
--> Provisioned cuncurrency 
--> SnapStart ** 

---


We can run a lambda function inside a vpc
make sure your lambda role has "AWSLambdaVPCAccessExecutionRole" policy.


## adding layres to lambda function
1. we need to give the folder name python and inside that we need to put the packages we want to use in our lambda function(Lambda automatically adds specific paths under /opt to Python's module search path (sys.path) and lambda fuction looks for modules in this path /opt/python). if you not give the folder name python then it will not work you will get the module not found error in lambda fuction.

```sh
pip install mysql -t python  # it will install the mysql package in python folder
zip -r layer.zip python  # it will create a zip file of python folder
```
upload the layer.zip file to lambda layer and then add that layer to your lambda function. Now you can use the mysql package in your lambda function.











