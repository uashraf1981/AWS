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

            ** Remember: You need to first create a role for SSM to run commands on EC2 instance and turn recording is on in 
            iunventory.

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

Is an important AWS security tool and monitors any suspicious activity within Windows or Linux EC2 instances. Opposed to AWS Config, which only looks at resources, AWS Inspector looks INSIDE Windows and Linux instances. It leverages an agent installed inside EC2 instances which monitors processes, network and other stuff using some pretty advanced analytics.

            AWS Inspector agent on EC2 --> AWS Inspector --> SNS
                 (Target)
                 
            Target: You can define either a specific EC2 instance, or you can point it to all EC2 instances. 
            Packages:
               - Common vulnerabilities and exposures
               - Center for Internet Security (CIS) benchmarks
               - Security Best Practices (Disable root login on SSH, Configure Password Max Age, Password Complexity)
               - Runtime Behavior Analysis (Insecure login, unsecured TCP listening ports, insecure root process permission)
               
               * RULE: A rule is a fundamental unit which checks for something specific in AWS Inspector. For instance, a 
               rule can check for a specific vulnerability or a particular OS for example. 
               
               * RULES PACKAGES: Rules packages contain a collection of similar rules which are grouped together.
               
               * ASSESSMENT TEMPLATE: Brings together all the elements i.e. specifies the targets (which EC2 instances are
               the targets and which rules packages will be applied). 
 
 Following steps guide us on how to configure AWS Inspector:
 
 The first step is to create an IAM role to attach the EC2 instance which would allow the System Manager (SSM) to install  
 the AWS Inspector agent onto the EC2 instances.
 
 1. Go to AWS Inspector
 2. Configure it to run weekly
 3. It will create a i) default target ii) default 
 4. Now we want to install AWS inspector agent on EC2 instance, but for that we first need to attach a new role to the EC2 
 instance, which is the Amazon EC2 role for SSM (Systems Manager) since we need SSM being able to push AWS Inspector agent 
 to EC2. So after we attach the IAM role SSMManagerEC2Access, we then go to Inspector->run->create run command -> configure 
 AWS package -> we should see the EC2 instance on this screen.
 There is a special run command which we need to use to be able to install AWS Inspector on EC2 using SSM.
 6. Now go to AWS Inspector and select targets which can be a specific EC2 instance or a collection of EC2 instances. We configure it for all EC2 instances and selecting "preview targets" and it will show all EC2 instances it has picked up.
 
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/assessmenttemplate.png)
 
 
 
 
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
            
TO RUN THE ASSESSMENT PROCESS
-----------------------------
Go to assessment template and select run and it will start running.
            
TO CHECK STATUS OF TARGETS AFTER RULES HAVE RUN
-----------------------------------------------
The results will be a list of vulnerabilities which have been ranked as high medium and low. You can also view the vulnerability result in JSON format.

You can automate the remediation process by providing AWS Inspector as the CloudWatch event source.

Difference between AWS Config and AWS Inspector

AWS Config looks at config changes at different AWS products   VS. Inspector matches internal workings based on rules (vulnerability report generation, looks for behavior it is a scanning engine and it can assess for a period of time e.g. any process or CPU behaving strangely)

AWS Systems Manager allows you to apply patches, update software, run commands.

            
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
            
 # LAB: AUTOMATIC RESOURCE REMEDIATION WITH AWS CONFIG
 
 Boto3 = AWS SDK for Python.
 
 ![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awsconfiglab.png)
 
 In this lab, we will create an AWS Config rule that will inspect security groups for unrestricted SSH access. We will also create a lambda rule to remediate the resource. In the figure shown, we want to make sure that only on-premise IPs can SSH into the EC2 instance which is on the public subnet, but we don't want anyone from a public IP SSH into the EC2 instance.
 
 As the figure shows, the security group allows SSH from public i.e. o.o.o.o/0 and our job is to create an AWS config rule that detects this, and a lambda rule that changes this non-compliance by modifying the security group to only allow traffic from the on-premise IPs i.e. 10.10.0.0/16.
 
Step 1 - In AWS Config, we want all resources to be recorded in this region.
Step 2 - We want all resource configurations to be stored in an S3 bucket.
Step 3 - Select an appropriate role, create one with read-only access for AWS config for your resources and this will also need to include permission to send notifications to S3 and SNS.
Step 4 - Next go to the rule, AWS has a list of pre-configured rules, wo we will select SSHRestrictedAccess.
Step 5 - When you do next and confirm the rule, AWS will search all of your resources for violation of this rule and discover the group with the violating rule. You can double check the group to be sure.
Step 6 - First we creaate an SNS topic that will send us notifications of violations.
Step 7 - Now we will create a lambda function, so head over to lambda and create a lambda function. We will create a function form scratch, name it remediateSG, choose Python 3.6 and choose an existing role if you've already defined it. You need to use the code from the github repository lambdafunction.py
Step 8 - Next we create the CloudWatch Event Rule which triggers the lambda function and SNS topic though I am not sure how that is connected to the AWS Config.

** Got it! basically you specify in the lambda rule the AWS Config rule that should be used to detect policy violation.

Step 9 - You will keep getting SNS notifications until the lambda corrects the violating rule automatically which it should immediately after detecting for the first time.

Next Lab in which we create an AWS Config rule that will detect problems in S3 bucket, we create a violating one.
next we will create the lamda which will remediate the public read access from the S3 bucket ACL.
Finally, we will create a scheudled rule in the CloudWatch to invoke that lambda rule.
Go to AWS Config, and create a rule and select S3 bucket public read access rule so that we can detect any public buckets.
Now create the lambda function and in the code of that, specify that use this particular AWS config rule.

# LAB: AUTOMATIC REMEDIATION OF AWS INSPECTOR FINDINGS IN AWS

 ![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awsinspectorlab.png)
 
 In this lab, we use the System Manager to remotely execute run commands on the EC2 instance which we would otherwise require SSHing into the instance.
 
 The inspector agent is running on the EC2 instance, which will scan for vulnerabilities. The inspector has a predefined set of rules which it uses to inspect your environment for vulnerabilities. 
 
 Interesting flow of events totally different from AWS Config or CloudWatch
 
            AWs Inspector -> SNS Topic -> Lambda Function -> Systems Manager -> Action to fix vulnerability e.g. yum update
            
 AWS labs has shared the code for this lambda function.
 
 The Lambda function contains the following:
 
 1. Findings for CVE vulnerability for AWS inspector
 2. Grab EC2 instance ID where issue was find
 3. Get meta data from SSM
 4. Upgrde the system 
 5. SSM runs command to upgrage the instance
 
 Most new EC2 instances has SSM agent installed by default.

# Troubleshooting CloudWatch Events

CloudWatch events is a workflow type of service where on one side you have the sources and on the other side, the targets.

CloudWatch Event Rules:

1. Scheduled rules:
      1.1. Scheduled rules
            1.1.1 Time interval based schdeuled rules e.g. every 5 minutes (usually straightforward)
            1.1.2 Cron expression based scheduled rules, lots of mistakes possible here so make use of docs and examples
      1.2 Event Pattern rules
            They are all about JSON. You can configure any service e.g. EC2 then select StateChangeNotification. You can 
            even specify a specific state e.g. when it starts and you get the JSON expression. Most people makle mistakes in 
            the JSON code. Better to click the graphs against this rule to see what is being captured.
            
            ** An important thing is that if you configure IAM calls in CloudWatch events, then they always come from us-
            east-1 regardless of what you have configured, so if those calls are not coming us-east-1, they will not be
            picked.
            
There are two main types of permissions in AWS:

1. You need to be given correct permission as a user to be able to access CloudWatch events (e.g. create rules, adjust them)
2. CloudWatch events need to be given permission to invoke the targets e.g. lambda or SNS. For each of these targets, these targets have attached policies which need to specify which resources are allowed to access it e.g. the SNS topic that we select, if we go to its policy, then we can see that every SNS topic has a policy attached to it which swpecifies which services e.g. CloudWatch are allowed to call upon it.

For lambda you need to create a role which uses the policy AWSLambdaBasicExecutionRole.

            ** Interestingly, lambda function role is automatically attached i.e. allows CloudWatch events to run lambda. If 
            you have edited this policy, then that maybe this is causing problems.
            
            ** similarly, putting encrypted messages into SQS will involve KMS which will again need permissions.
            
            ******* CLOUDWWATACH events will delivery you at least one copy of the event, but it can also send duplicates
            somtimes i.e. at least once delivery model which ensures message is delivered, but not exactly once so keep that 
            in mind in the real-world.
            
            EXAM QUESTION: For certain type of events e.g. S3 object events in CloudWatch events, there needs to a 
            CloudTrail that is actively logging data events since CloudTrail has two options, either log management level 
            events or data level events. This question comes in exams sometimes. So if you are using any data level events 
            in CloudWatch events then data events MUST be enabled in CloudTrail otherwise the events will not be triggered.
            active that is logging data level 
            
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatchnew.png)

# CloudTrail

Any action that you take in an AWS Account, through console, CLI or SDK is logged by CloudTrail i.e. any kind of API call. These events are stored in something called Event History and these are stored by default for 90 days. It is pretty much static list of events that you can filter, search and download in CSV and JSON format, but that's it, you cannot interact with any of the AWS services.

            CloudTrail provides two advantages:
            
            1) IT allows you to select the way CloudTrail records API calls e.g. read-only write-only
            2) Allows you to specify the destination for those logs e.g. a destination S3 buckets
            
            You can specify region based event logging or global events e.g. IAM events. 
            
            ** Exam: If you have selected to apply to all regions and a new region launches then it will apply to that also.
            
            * By default, CloudTrail only logs management style events i.e. events on the control plane e.g. new user 
            created. BUT you could also log data events and this is particularly important if you want things like e.g. 
            object events in S3 buckets. If data events are not enabled here then you could configuring whatever events
            you want in CloudWatch but they would not be triggered.
            
            * Remember to enable data API calls for both S3 and lambda it's just a good idea.
            
            * Encryption: By default, all CloudTrail API logs are delivered into the S3 bucket encrypted using S3 server-
            side encryption, although you could probably change it.
            
            * you could also configure an SNS topic to deliver notification emails against events. This warrants the 
            question that why do we need CloudWatch triggers, well the answer is that event delivery to bucket in CloudTrail
            is not real-time, so any triggers would not be real-time.
            
            ** EXAM: All CloudTrail events are actually recorded as JSON structures.
            
            The name of the folder is always AWSLogs -> CloudTrail, and there is a second folder CloudTrail-Digest which 
            ensures the integrity of the files. The folder contains sub-folders according to different regions.
            
            CloudTrail Log in S3 = JSON file with multiple CloudTrail Events = JSON structures within that JSON file.
            
            ** EXAM: CloudTrail log files sanctity is gone if you even move or rename or do anything with them.
            
                        CloudTrail offers two types of integrity checks:
                        
                        1. It delivers a SHA-256 hash for every file it delivers, making it impossible to modify
                        2. Dlivers a file every hour which references logs that have been delivered in the same hour i.e.
                        CloudTrail log deletions are automatically detected by CloudTrail
            
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudtrail.png)

The validation of logs is always done using the command line:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/validatelogs.png)
            
# CloudWatch Logs

Central hub for all types of logs in the AWS ecosystem.

1. We will connect CloudTrail to CloudWatch logs, so we will use an existing trail or create one then in the options, apart from delivery to S3, we can also configure to setup with CloudWatch logs, you just need to specify the name of the log group within which CloudTrail will be delivering the events.

2. We will need permissions for CloudTrail to able to ingest logs into CloudWatch logs, you can do this from within this section and a default role is created which allow CloudTrail to create log stream and ingest data into CloudWatch logs.

3. Now head over to CloudWatch logs and in the log groups, you will see the log stream that is coming from CloudTrail. You will see events within this screen. You can filter these CloudTrail events. These events can be unstructured, but generally, they are again JSON structures.

            Log Streams: Collection of events from a same or similar servers
            
            Log Groups: Collection of log streams
            
            Log Group contains -> multiple log streams, which in turn contain --> multiple events
            
            ** Metric Filters: Metric filters in CloudWatch logs, allow searches to be done on log groups. It is very 
            flexible and allows for searching for anything particular e.g. failed logins etc.
            
            ** EXAM: You can configure expiry for log groups, but you cannot set expiry for individual streams within group.
            
            Subscription: Is a system that allows you to stream logs from CloudWatch Logs to Kinesis or Lambda.
            
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatch2.png)

            ** REMEMBER, this chain of API call -> CloudTrail -> CloudWatch Logs -> Notifications is not real-time, if you
            want that, then use CloudWatch Events.
            
# CloudWatch Logs: VPC Flow Logs

            ** EXAM: VPC Flow Logs only capture meta-data about network traffic e.g packet src/dest ips ports etc, not the 
            payloads.
            
* VPC flow logs are automatically ingested into CloudWatch flow logs.

CAPTURE POINTS: The important concept here is that you can define capture points which are basically points on which you define to capture VPC traffic meta-data. So, if you define a capture point on the level of the VPC, then you can capture all the traffic coming to or from VPC in all the subnets for all the machines. Interestingly, any traffic between subnets will also be captured by this capture point.

VPC Level Capture point: Capture all the traffic at VOC level i.e. to/from all subnets and all machines.
Subnet Level Capture Points: Capture traffic going to/from subnet and within subnet
EC2 Level Capture Point: Onlyh traffic to/from that machine.

Step 1 - Go to VPC dashboard
Step 2 - Create a flow log, within that, you need to select a filter (all, accepted or rejected traffic)
Step 3 - Within that screen, select an appropriate role to give permissions
Step 4 - Go to CloudWatch and create a log group
Step 5 - Select that created log group as the destination log group here in VPC Flow logs.

            * Exam: Remember, VPC Flow Logs are not real-time. If you need real-time traffic, you need to use packet traffic 
            analysis tools on the instances.
            Exam: Traffic to/from Amazon DNS servers is not logged as a policy matter.
            Exam: Windows activation traffic is also no logged as a policy matter.
            Exam: Traffic to/from meta-data ip address 169.254.254.254 is not logged as a policy matter.
            Exam: Traffic to/from DHCP is not logged as a policy.
            Exam: Traffic to/from VPC router is no logged as a policy.
            Exam: Traffic from the DNS service i.e. Route53 is not logged.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/capturepoints.png)

# CloudWatch Agent for EC2

The CloudWatch Agent can run on EC2 instances and even on the on-prem servers.
* They feed metrics that are normally never available to CloudWatch such as system level metrics such as CPU and memory usage and application logs. Now these become available to the CloudWatch.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatchagent.png)

Step - 1: First thing you need to do is to create an IAM role, select that EC2 instance will assume this role, then attach two policies 1st: CloudWatchAgentServerPolicy (to interact with CloudWatch) and 2nd: AMazonEC2RoleForSSM (to interact with SystemManager which we will use to automatically install agent on EC2 instances). 

Step - 2: Go to the EC2 instance and attach the IAM role.

Step - 3: Now we will install the CloudWatch agent manually on EC2. Download using wget then use config-wizard which will ask which OS and which metrics to be collected etc. Any OS config files that you want yes we want two log files:

   1. var/log/messages then select log group name.
   2. var/log/secure

Step - 4: You need to attach an additional policy SSMFullAccess to EC2 for Step - 3.

Step - 5: Now if you go to CloudWatch, you will start seeing two new log groups "mesages" and "secure". Inside these message groups, we will see the logstream.

The process so far was manual, now we do it automatically, we will use parameter store.

Step - 6: Download using wget, then unzip like before, install with sudo permissions,

Step - 7: We will issue a command which is the config that we stored in the paremeter store previously.

The third way is that instead of running anything from the instances, we will go to instance manager.

We will go to System Manager and run:

            Document name prefix: Equal : AWS-ConfigAWSPackage
            
then select target: EC2 instance
Interestingly, it also allows you to selec the target -> CloudWatch Log group

You can use system manager to deploy on hundreds of instances.

We are not done, we also need to run AWS-CloudWatchManage-Agent command on System Manager, target agai EC2 then again target to CloudWatch. This is going to retrieve from the parameter store.

            Parameter Store is where to store configurations securely in AWS.
            
# CloudWatch DNS Logs
DNS is a very important service.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/DNS.png)

We can have Route53 as the registrar for this domain.
            
            * In order to log wuery logging for the domain, you only need AWS to be the name resolver, but it does not 
            have to be hosted by AWS.
           
We now create a sub-domain and 

Set Query Logging:

You need to select the hosted domain name -> Configure Query Logging -> Specify the CloudWatch log group. Select where the query logs will go e.g. new log group. Create Log group. 

We also need permissions, so we will create a new resource policy, so we do this within this same window.

            * Remember, since DNS is a global service so it will take some time to propagate the query logging to all the 
            global edge locations.
            * In the log groups -> log streams of the CloudWatch will show separate edge location based streams containing
            the pings etc. done and the name will be based on the nearest airport where that edge location is located e.g.
            AMS will be for Amsterdam.
            * It is not real-time. just a basic logging service and gives just a general state.
            * Exam: These logs are NOT available for provaye domains i.e. the resolver MUST be Route53, if you are using
            a third party as resolver then it will not work.... hosting the domain does not matter.

# S3 Access Logs

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/s3logs.png)

Basic idea is that the logs hosted by CloudWatch i.e. CloudWatch logs can also be stored in S3. We want to offload this load onto S3 which offers both short term and long-term storage. It is a much more durable place for storing the logs and by sending it to another account (restricted to a small number of selected users), we can move it away from the blast radius of our account. The logic is that if the account gets compromised then the CloudWatch logs can also get compromised,

            CloudWatch -> Logs -> LogGroups -> VPC Flow Logs -> Export to S3 then select the bucket in this or other acct.

Now S3 comes with its own logging since it offers a variety of services such as hosting a static websites, or storing contents in buckets, so it makes sense that it has its own logs.

            S3 logs track:
             - Object accesses (requester, bucket name, time, request action, response status)
             
             S3 -> bucket -> Create additional bucket (to store logs) ->
