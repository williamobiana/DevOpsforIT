# Store Application Log Files with Amazon CloudWatch
When you have to scale up your system on a cloud environment like AWS, you’ll probably have to store the log files produced by the operating system and applications in a cloud-based storage solution. But what do you do when instances come and go? And how can you keep up with expiration of old log files?

This is where AWS cloudwatch comes in. you can store your custom log files to cloudwatch and also monitor the incoming log entries to veiw as cloudwatch metrics. (example: monitoring your web log files for 404 erros)

## Getting familiar with the terminology
Here are some terms we need to understand, because you need to understand in order to use cloudwatch

* LogEvent: activity recorded by the application or resource being monitored
* LogStream: sequence of Log Events from the same source/resource
* LogGroup: group of Log Streams that share the same properties, policies, and access controls
* MetricFilters: filters that tell CloudWatch how to extract metric observations from events
* RetentionPolicies: policies that determine how long events are retained
* LogAgent: agents on your EC2 instances that store Log Events in CloudWatch

## Getting started with cloudwatch logs

* install the LogAgent and congfiure to capture logevent
* view the log groups
* view the log stream
* define the metric filter

### Install the LogAgent and configure to capture logevent
We can install the cloudwatch agent on an ec2 instance using the AWS documented steps [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/QuickStartEC2Instance.html)


### View the log groups and log stream
on the cloudwatch page, select the log groups section and view the log stream

### Define the log metrics
on the cloudwatch page, select the log metric section and define the query for the metrics you want to track and run.
