# Domain 2 - Logging and Monitoring

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/s3alertsoverview.png)

Interesting RRSObjectLost event was generated in S3 buckets, when an object got moved to Reduced Redundancy Storage (RRS) but got deleted during the process due to hardware issues. Historically, cloud developers made sure that such an event triggered lambda or other mechanisms to resotre that particular object back.

Important: If you want to trigger something based on S3 events e.g. SNS notification (email) on an object delete, then you MUST configure permissions to allow S3 to notify SNS for example. Important from exam perspective. So basically, you will go to that particular SNS topic, edit it's policy and in the advanced editor, you are going to add permission for S3 notification.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/snspolicy.png)

So now we can go to S3 -> Bucket -> Properties -> Events -> Notifications -> SNS it wil work now.

# LAB: Create a workflow for making any object uploaded to a bucket converted to being private

Step - 1 Create a lambda role for S3 bucket so we make the following policy and attach to this role:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/lambdapolicy.png)

Step - 2 Next, we create the lambda function itself and paste our own custom code in Python for this purpose.

Step - 3 Now we go to the bucket and create event notification and configure it to be sent to lambda and save.

Then you can upload a public object to this bucket and check CloudWatch logs to see how this panned out.

      Relevant Files:
      
     lambda.py
     lambda_policy.json
     
CloudWatch Logs: Monitoring and Alerting
----------------------------------------

CloudWatch is AWS monitoring, logging and alerting tool. It acts as a central hub for monitoring, app logging, app monitoring, from security perspective, it allows for monitoring and logging to be separated from the actual objects. This is important as exploited objects do not impact the logging.

The types of things that CloudWatch and CloudWatch logs monitor:

1. Account and product level data and logs - CPU utilization of EC2, network throughput of EBS, execution logs of lambda. It receives data that you have no interaction with, it receives data continuously.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatchlogs.png)

Custom Metric Filters
---------------------

AWS contains some pre-defined custom metrics, but you can also create custom metrics. You can make your life very simple. Just check the CloudWatch logs for your concerned stuff, then select any of the specific strings that are output. For instance, you can monitor the S3 bucket upload logs where we monitored if an object is not private. We see that the word "private!" is there in the logs. So you can create a custom filter using this keyword as the matching pattern.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/customfilter.png)


Then we can create a custom alarm based on this metric filter. Then we can check the alarm in CloudWatch has gone to an alarm state or not.

            Alarms can be used to trigger many things e.g. alarms and metrics and metric filters can be used to lots of 
            stuff in security e.g. trigger auto-scaling.
            
CloudWatch Events
-----------------

CloudWatch is the hub of events in AWS. It allows event-driven security. CloudWatch events are just like alarms but instead of thresholds, they perform pattern matching based on metric filters. They are like the central hub of events.

            CloudWatch Events -> Provide event-driven security within the cloud ecosystem
            CloudWatch Events -> Sits as a hub within the AWS ecosystem i.e. between the products
            
The input to CloudWatch events are different sources such as EC2 instance being started, an IAM user being created etc. Every event is described by a JSON document.

So basically in AWS, the flow of events is as follows:

            *** IMPORTANT GLOBAL PERSPECTIVE ON EVENTS ***
            
            Event (from EC2, IAM etc) -> CloudTrail -> CloudWatch Events -> Outputs (SNS, SQS)

# LAB: CloudWatch Events

Step - 1 Go to CloudWatch -> Events -> Create Rule -> (Pattern or Schedule)
                                               Schedule -> periodic/cron expression & means CloudWatch generating new events
                                               Pattern -> means cloudwatch events is listening for events from other sources
                                                      
                              Remember that when creating CloudWatch events, you must select the:
                              
                              service    -> The originating service e.g. EC2, or IAM
                              Event type -> the specific event, usually it is CloudTrail API call related to that source
                              
Step - 2 For instance, we select EC2 instances and then select All Events then anything that happens will be captured. So for instance, we want to make sure that our servers are always up and running.

Step - 3 Now this requirement of lambda needs a role, so we will create a lambda role as follows:
         IAM -> create role -> AWS services -> Lambda -> services -> select the policy -> Create new policy -> Visual Editor -> service EC2 -> Access Level -> Write Actions -> Start Instances -> done -> resources -> all resources -> name policy -> create policy -> 2 permissions (AWSLambdaExecutionRule and start instances permission) -> then we go back to the role and attach this policy to that lamda role
         
Step - 4 Now create the actual lamda function itself. 
                             
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/lamda2.png)

Step - 5 Now we go back to CloudWatch Events -> Create a rule -> Service Type (EC2) -> EC2 instance state change notification -> specific state (stopped) -> then map to 2 targets (lambda function and SNS topic)

                  ** Important **
                  If you want to create a Cloudwatch rule that relates to data or API calls such as object level changes in 
                  S3, then AWS requires that you must have a CloudTrail with "data events turned on".
                  

                  ** Important **
                  Remember that to call AWS CloudWatch your system needs to have access to the public Internet since 
                  CloudWatch has public endpoints.
                  
                  Exam tip: There is an exception, you can use VPC end point to access CloudWatch even from private network.

# Multiple Accounts: CloudWatch Event Buses

CloudWatch event buses can accept events from other AWS accounts. 

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatchbuses.png)

Source account event rule -> receiving account event rule -> SNS

Step 1 - We will create an SNS topic and subscribe an email address.

Step 2 - The procedure is fairly simple, we create one account as the receiving acccount, add permissions to other accounts and receive all notifications in this account. So just simply collect account IDs for all the account that you want to add, then go to Cloudwatch in the receiving account, then go to event buses and then add permissions i.e. give IDs of the trusted accounts i.e. account which will send events to this account.

Step 3 - Now get the account ID of this account and then go to the trusted account, then create an event rule in that account e.g. IAM -> Add User

Step 4 - Select target as event bus and select the receiving account i.e. provide ID of that account.

Step 5 - Next we go back to the receiving account and create an event rule which sends SNS. Remember to create the rule same as in the source account i.e. service: IAM and event API call and CreateUser event.

Step 6 - Go to the source account and create a new user to see if it works.



