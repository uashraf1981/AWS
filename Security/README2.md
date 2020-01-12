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








