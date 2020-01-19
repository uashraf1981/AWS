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

            ** Important **
            1. We need to add explicit permissions in the receiving account for the account IDs that are permitted to send.
            2. Not a good idea to allow everyone otherwise, we need to go to the JSON rule and add rule by rule persmission
            3. Remember, the source rules and receiving account should be in the same account.
            4. You can send ALL events from the source accounts into a single account and then only define rules in that 
            account i.e. instead of creating rules in every child account, you filter only at the central account.


# AWS Config

Tracks configuration of all AWS resources. It helps with compliance standards and allows us help in audit. It allows monitoring of all AWS resources and allows for continuous monitoring of changes. It records what changed and who chanhged it. So continuous compliance. What security standards your organization has and it helps manage a consistent record of changes. Also highlights unauthorized changes.p

Also helps in unauthorized changes.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awsconfig.png)

            AWS Config has two essential components:
            
            1. Configuration recorder -> constantly records the configuration of all resources i.e. tracks every change in          
            every resource in every region. AWS config is enabled on a per-region basis, it can operate in any region, but 
            it is not global, so you need to enable it. It needs a role to get appropriate permissions to record the 
            configurations of all resources. 
            
Setting up AWS Config
---------------------
The starting configuration is pretty simple. 
1. Inform the config tool as to what resources you want to monitor. The default is to record all resources in all regions
2. You need to provide an S3 bucket where it will store all this information
3. You can optionally link it to an SNS topic on this same screen and it will stream any configuraiton changhes in the monitored resources to that SNS topic
4. AWS Config Role: You can select the option "Create a role"
5. As soon as the role is created, it starts tracking all the resources

            Configuration Record -> State of a particular resource at a particular instance of time e.g. it can be a          
            security group, then what rules are in that security group, what ports those rules allow, what's the group name 
            and so on. If it is an EC2 instance, what is the size, what disks are attached, which security groups are 
            attached. 

            Configuration History -> a record of every single change of that resource.
            
            Configuration Stream -> SNS topic to which we can send the changhes.
            
            
            2. Config Rule (works as a Compliance Checker) -> Ensures organization wide compliance to something. For 
            instance, you can define that what ports should a security allow to be open, then you can apply that rule to all 
            the security groups in the organization. So if a security group only had good rules, it would be marked as 
            compliant whereas if it only had bad rules, then it would be marked as non-compliant AND more interestingly, you
            can define actions that you can take if a resource has been marked as being non-compliant and that action can
            remediate the non-compliance. AWS helps us by providing a lot of pre-defined rules such as CloudTrail is 
            enabled, database backups are enabled, detailed monitoring is enabled on EC2 instances.
            
            Remember that some rules in AWS config should only be triggered based on change, while in some situations  some
            rules should be triggered periodically. 
            
            * Normally these rules are triggered automatically after every change either in real-time or we can manually 
            trigger this rule as well.
            
            ![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awsconfigrulechange.png)
            
            * Another interesting thing about AWS Config is that it is storing relationships as well e.g. if a new rule for 
            opening port was added to a security group, it will show all the EC2 instances and VPCs that this new security 
            group belongs to.
            
            * Another interesting thing about AWS Config is that it stores the history of the resource for all the changes.
            
            ** I think that main difference between AWS Config and AWS CloudTrail is that AWS config stores information 
            about any configuration changes in the AWS ecosystem (including creation, deletions and modifications) whereas 
            AWS Config stores information about any API calls made between resources. Generally speaking, AWS Config is an 
            account-level resource monitoring thing, it does not look inside resources e.g. not what happened inside an EC2 
            instance.
            
            ** AWS Config is basically limited to the region that it is defined for, but you can create aggregate AWS config 
            in all regions to get a unified information.
            
# AWS Inspector

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awsinspector.png)

Is an important AWS security tool and monitors any suspicious activity within Windows or Linux EC2 instances. It looks INSIDE Windows and Linux instances. It leverages an agent installed inside EC2 instances which monitors processes, network and other stuff.

            AWS Inspector agent on EC2 --> AWS Inspector --> SNS
                 (Target)
                 
            Target: You can define either a specific EC2 instance, or you can point it to all EC2 instances. 
            Packages:
               - Common vulnerabilities and exposures
               - Center for Internet Security (CIS) benchmarks
               - Security Best Practices (Disable root login on SSH, Configure Password Max Age, Password Complexity)
               - Runtime Behavior Analysis (Insecure login, unsecured TCP listening ports, insecure root process permission)
 
 Following steps guide us on how to configure AWS Inspector:
 
 1. Go to AWS Inspector
 2. Configure it to run weekly
 3. It will create a i) default target ii) default 
 4. Now we want to install AWS inspector agent on EC2 instance, but for that we first need to attach a new role to the EC2 instance, which is the Amazon EC2 role for SSM (Systems Manager) since we need SSM being able to push AWS Inspector agent to EC2. 
 5. There is a special run command which we need to use to be able to install AWS Inspector on EC2 using SSM.
 6. Now go to AWS Inspector and select targets which can be a specific EC2 instance or a collection of EC2 instances.
 7. 
 
 # Now Going to Do a Short Cut Training
 
 1. AWS Inspector
 2. Web Application Firewall and AWS Shield (lab: Blocking Web Traffic with WAF in AWS)
 3. Security Groups
 4. Host Based IDS/IPS
 5. IAM Policies
 6. Identity Federation
 7. Key Management System
 8. Cloud HSM
 
 DevSecOps Essentials
               
            
 # AWS Web Application Firewall (WAF) and AWS Shield
 
Remember that WAS sits in front of a CloudFront distribution or Application Load Balancer. It is a layer 7 product and can watch traffic both globally as well as look for patterns INSIDE the traffic such as SQL injections.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/waf.png)

It is a relatively simple product and its base entity is Web ACLs which are Web Access Control Lists i.e. made up of rules and each rules can have multiple conditions. There are two things that you need to understand i.e. conditions and rules.

Web ACL:
Conditions --> Traffic matching a geo-tag e.g. traffic coming from Australian IP addresses
Rule --> Is triggered on one or multiple conditions
 
           
            There are two types of rules:
            
            1. Regular rules that apply to cloud traffic
            2. Rate based rules that apply to conditions e.g. a condition violated a number of times e.g. strange traffic 
            coming from a certain IP range.
            
So conditions match things, e.g. coming from certain IP addresses, or SQL injection signatures.

Conditions         ---bind to--->               Rules              ---bind to --->   Web ACL 
(suspicious IPs)                   (Match suspicious IP and SQL)                  (Matches Rule 1 or Rule 2)
(SQL)



Conditions     ->.  Rule contain muiltiple conditions combined in one rule  -> Web ACL contain multiple rules combined.

            ** Remember you can default allow or default deny Web ACL.
            
# Security Groups

Security groups are always created inside a VPC. So you cannot associate a security group that is in one VPC to a resource that is in another VPC.

            ** Important technical point: Security groups are not technially associated directly with EC2 instances, instead 
            technically speaking, they are associated with the network interface card of the EC2 instance.
            
            Think of security groups like an interface or barrier in front of the network interface card.
            
            * Security groups are stateful.
            
            * Security groups are default deny i.e. you can only add rules to ALLOW traffic, DENIAL is otherwise understood, 
            there is actually a default security group that gets hit that denies everything. So you cannot explicitly deny 
            traffic in security groups.
            
            ** Security groups can also reference other security groups i.e. allow traffic from other security groups.
            Security group can also reference itself e.g. atatch security group to multiple EC2 instances and allow traffic 
            between them i.e. cvool feature that you can create grouping of EC2 instances which can talk to each other on a 
            specific port.
