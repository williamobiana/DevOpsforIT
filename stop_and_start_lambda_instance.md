# Start & Stop instance with AWS lambda and eventbridge

## Prerequisites
Launch an ec2 instance to get the instance_ID

## Steps:
1. create custome IAM policy and execution role for the lambda function.
2. create lambda function that will start and stop the instance.
3. test the lambda function.
4. create eventbridge rules that trigger the function.

## Create custom IAM policy and execution role for the lambda function
* sign-in to the AWS console
* go to IAM > policy > create a new policy
copy and paste the following JSON policy document into the policy editor

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Start*",
        "ec2:Stop*"
      ],
      "Resource": "*"
    }
  ]
}
```

* create IAM role and use the permission policy you just created

## Create lambda function that will start and stop the instance
* go to lambda > create function > author from scratch
* under basic information, add the folloing:

function name: StopEC2Instances\
runtime: python 3.7\
permission: change default execution role\ 
execution role: use an existing role\
existing role: choose the IAM role you created\
create function

* under the code tab > code source, copy and paste the code below into the editor.
this code stops the ec2 instance

```
import boto3
region = 'us-west-1'
instances = ['i-12345cb6de4f78g9h', 'i-08ce9b2d7eccf6d26']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print('stopped your instances: ' + str(instances))
```
*Note:*\
replace region with the region your instance is running in.
replace the instance_ID with your running instance_ID.

* deploy

* go to the configuration tab > general configuration > edit
* set timeout to 10 secs ( this gives the instance enough room to boot)
save

## Test the lambda function
* return to the code tab > code source, and * select test
* test event opens a dialog box, create new test event
* enter event name and create
* select test again to run the function
* check the status of the ec2 instance to confirm that the function is working.

## Repeat the function configuration for start instance
function name: StartEC2Instances\
* paste the code below to start the ec2 instance 

```
import boto3
region = 'us-west-1'
instances = ['i-12345cb6de4f78g9h', 'i-08ce9b2d7eccf6d26']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.start_instances(InstanceIds=instances)
    print('started your instances: ' + str(instances))
```

* test the lambda function

## Create eventbridge rules that trigger the function
* go to eventbridge > create rule 
* name of rule: StopEC2Instances
* description: optional
* define a pattern: schedule
* select cron expression (use the [cron expression syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) to pick a time)
* select targets: choose lambda function
function: choose function that stops your instance
create

## Repeat eventbridge rules for StartEC2Instances
name of rule: StartEC2Instances\
function: choose function that starts your instance
