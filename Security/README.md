# Domain 1 - Incident Response


Incident has three sub-domains:

1. AWS Abuse Notice
-------------------
AWS teams have automated monitoring tools with which they are constantly monitoring the AWS infrastructure for things like:

    - large changes in patterns, port usage in pattern with known exploits, penetration testing 
    - inspect things internally i.e. attacks againt AWS hypervisors or other internal virtual network infratructure
    - inspect outgoing traffic from your environment e.g. are you doing something illegal or your resource has been hacked and 
    is now being used by someone else
    - manual abuse notices in which someone informs AWS team that your resources are being used illegally
    
Acceptable Use Policy:

a) Cannot use AWS for illegal, harmful, infringement or offensive material
b) Cannot use AWS for sniffing, falsisying origins
c) No Network Abuse - no monitoring and crawling, denial of service attacks, operting open relays an proxys
d) Cannot use AWS for any mass or unsolicited emails or messages

Penetration Testing:

AWS allows penetration testing, load simulation etc but you need to get authorization first.
                  
                  Three main types of events that require prior permission from AWS:
                  
                  i) Simulated Event Testing - e.g. DR simulation, load testing, security training to team, red/blue teaming
                     you just need to send an email to AWS regarding the event
                     
                  ii) Vulerability and Penetration Testing - A web form that you need to fill in and it is pretty expensive 
                  e.g. who you are, contact, systems or targets that you are attempting te. gateways, DNS style attacks, 
                  details of where the pen testing are coming from, IP address who owns those and contact. Overview of what 
                  to expet e.g. start-end dates, who will stop the testing (e.g. if AWS want you to stop the test). AWS only 
                  allows pentests against certain services e.g. EC2, RDS, AURORA, CloudFront, API Gateway, Lambda, 
                  Lightsail, DNS zone walking. Even within these, there are further restrictions e.g. not allowed to 
                  pentesting against EC2 and RDS not allowed to test for small instance types e.g. t instance as these are 
                  always over subscribed and thus your testing can affect other customers.

Four reasons why you may get abuse notice:

                  1) Compromised instance - EC2 may have been compronised and become part of botnet.
                  2) Secondary Abuse - a malware got installed and is calling back home
                  3) Application Function - a genuine app function may get alerted , it may be your genuine app reaching 
                  out to the Internet, but for the automated monitoring tool this may appear as an attack.
                  4) False complaints - You just get false complaints and other AWS users may report. You may have a web 
                  server and it is talking to another server in someone else's resources in their VPC by accident
                  
AWS Abuse Notice:

1. You must respond to the abuse notice
2. You must remove the immedite exploit that has occurred e.g. account credentials being leaked.

                  Steps to Take to Respond to Abuse Notifications - basically be paranoid and risk averse:
                  
                  1. Change the root password and the password to all IAM users.
                  
                  2. Add MFA to all admin users and others who have acces to the console.
                  
                  3. Generate new key pairs for all the EC2 instances by deleting the old key pairs, relaunching the AMI as 
                  a new EC2 instance and edit the .ssh/authorized_keys file which gets populated with the key at the boot   
                  time. AWS uses 2048-bit RSA based public key cryptrography and stores only the public key part. When 
                  launching an instance, you specify the key pair or generate a new key pair. The public key part of this 
                  key gets stored in the .ssh/authorized_keys file on the instance.
                  
                  Important: AWS DO NOT store the private key pairs so if you lose the private key for an instance-store-
                  backed instance, then you need to terminate the instance and launch a new one, and you will loose 
                  data. Interestingly, if however, you loose the keys to an EBS-backed instance, then you still recover it.
                  
                  Important: If you have multiple users which access an EC2 instance, then you can generate their key pairs, 
                  give them their private part of the key and store the public keys in the .ssh/authorized_keys file.
                  
                  4. Delete or rotate compromised IAM keys.
                  
                  5. Delete the following:
                   - unrecognized instances
                   - spot bids
                   - IAM users
                   
                   6. Contact AWS support.
                   
                   
                   Pro tips:
                   
                   Smart idea is to run dynamic resources and users that are active.
                   Smart idea is to use the root account just to generate IAM admin user.
                   Smart idea to use pwd policy to insist on using strong password.
                   Smart idea is to always use IAM roles.
                   Smart idea is to always rotate credentials.

Git Secrets
-----------
Since bots are always scanning public git repositories for stuff like EC2 credentials or access keys, therefore Github has released a tool called Git-Secret which if you install with the git uploading tool, will not allow you to accidentally publish passwords and secret keys for EC2 instances. There are also tools available which you can use to scan your own public repositories to show your leaked public access keys.
                  
It is pretty common for instances to get compromised through leaked passwords or keys. Often this is due to committing keys to a public repository like github. There are bots that are always scanning public github repositories.
                  

# LAB: Audit a Source Code Security Scan Using Git-Secrets in AWS
Our task is to audit a Github source code repository to scan for vulnerabilities. We will use AWS lab's "git-secrets" to perform this scan. 

The steps that we will follow include:s

Create-EC2 -> Clone git repository -> Install git-secrets -> Install Git Hook -> Scan Repository -> Identify Code for Fix

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/gitsecrets.png)

Step 1 - EC2 has already been created by the platform.
Step 2 - SSH into our instance using the given credentials.
Step 3 - Install git using the yum manager.

            sudo yum install git
            You can use the -y flag to simulate automatic yes to all queries raised during installation.
            
Step 4 - Clone the git repository using the following command:

            git clone https://github.com/linuxacademy/la-aws-security_specialty
            
Step 5 - Clone the git-secrets repository using the following command:

            git clone https://github.com/awslabs/git-secrets
            
Step 6 - Install the git-secrets using the following command:

            cd git-secrets
            sudo make install
            
Step 7 - Go to the repository that we want to scan.

Step 8 - Install git hooks which are basically a number of rules that can be applied to a git repository. In our case, we want to apply a series of AWS best practice rules. These are AWS patterns and will scan for AWS Access Key IDs, Secret Access Keys, Account IDs, or other patterns defined by AWS. These will catch most credentials leaks but no guarantee we should go due diligence.

            sudo git-secrets --register-aws
            
Step 9 - Now scan the repository using the following command:

            git-secrets --scan
            
 ![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/vulncode.png)    
 
 Step 10 - Git-secrets also proposes some mitigation strategies, but this information is passed onto the developer team.
 
 

# 2. AWS and the End-to-End EnIncident Response Framework

Preparation Phase
-----------------
Preparation -> Identification -> Containment -> Investigation -> Eradication -> Recovery -> Follow-up

* In the preparation phase, we can limit the blast radius by creating accounts within our main AWS account with different permissions in order to architect a system with segregated security.

        We use a "Service Control Policy" to limit what the child accounts within the root account can do.
        We can also use different VPCs to separate the services (e.g. EC2s) at the infrastructure level.

The only limitation of using VPCs in different regions is that you are only protected against network attacks. If an account exploit occurs such that, that account has access to multiple VPCs, then the infrastructure level protection means nothing  even if they are in different regions.

        We need to log everything to a central repository and encrypt it to protect as it may contain sensitive info.
        
        In AWS: Logs -> Events -> Alerts -> Response

The following are the products that assist in logging in AWS:

        AWS Services Used for Logging:
        
        CloudTrail    -> API Calls
        VPC Flow Logs -> Log traffic meta data coming into and going out of VPC
        EC2 instances -> By installing a cloudwatch agent and centrally collect app and OS logs
        S3            -> Can host data and has its own logging system
        CloudWatch    -> Allows to manage everything centrally
        AWS CSonfig   -> Perform configuration and compliance checks to see if any config or compliance changed
        
        * Lamda helps in automating response.
        
        
Remember there are two types of encryption: client-side encrpytion and server-side encryption.

We need to ensure that data is encrypted both in transit and at rest:

        AWS Services Used for Encryption:
        
        KMS -> Key Management Service Separation encryption/decryption away from storage
        S3 ->  Its own encryption/decryption technologies
        Certificate Manager -> Used for encryption/decryption for data in transit used for web client, load balancers
        Route 53  -> Also plays a part in encryption/decryption

Detection Phase
---------------------------------
We detect incidents, the most important and hardest of the incident response lifecycle:

        AWS services used in the detection phase:
        
        AWS CloudWatch -> Loggin and monitoring of metrics of CPU usage, disk throughput, application and system logs
        S3 events + lambda -> To detect upload and download of files
        
Containment Phase
-----------------
Containment is heavily dependent on the detection phase as you can't contain what you don't detect.

        Actions that you could take for containment:
        
        i) Take snapshots for offline investigastion
        ii) Stopping instances
        iii) Disabling KMS encryption keys
        iv) Change Route53 record sets
        
Investigation Phase
-------------------
In this investigation phase, you will be doing two main things:

1) Establish a timeline as to what happened exactly when
2) Inveswtigate the event and correlate things to establish what exactly happened and the chain of events

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/timeline.png)

You can do two types of firensics:

1) Live box forensics -> investigate the EC2 which was actually exploited
2) Dead box forensics -> investigate offline using snapshots in a safe sandbox environment

Questions to answer:

a) What happened and where
b) Where did the attack come from
c) What was the ingress - what was exploited e.g. which service was exploited

Eradication Phase
-----------------
Single objective to remove all the infections and resources from our environment.

        Major steps to do in the eradication phase:
        
        1. Delete and disable any KMS keys
        2. For EBS volumes, delete any infected files, create a new volume and copy the good files
        3. For S3 with S3 encryption-> delete the objects
        4. For S3 with Customer Managed Keys (CMKs), delete the object and the associated CMKs
        5. Secure wipe any affected files
        6. If the EBS volume was not encrypted then better to generate a new voume and avoid just sanitizing it
        
Recovery Phase
--------------
Recover resources one by one and monitor, monitor and monitor.

Follow-up Phase
---------------
For this phase, you can use testing and simulations and improving team efficiency is improvement.

# Configuration of Automated Alerting

CloudWatch-Logs accepts data from all the different services in AWS. However, remember that the CloudWatch-Logs is a small product within the bigger product of CloudWatch which can accept many traditional metrics such as CPU usage, network throughput and many others.

        CloudWatch-Logs -> CloudWatch Event Rules -> (Lambda Function, Systems Manager, SNS Topic, SQS)
        
        A very interesting alternate is:
        
        CloudWatch-Logs -> Metric Filters and Alarms on set of Logs to identify patterns -> Alarm generated
        
 Difference between the above two approaches are there in terms of how real-time they are.
 
        Two different architectures are possible:
        
IAM user creation -> CloudWatch-Logs -> CloudWatch Event Rules-> SNS Topic
        
                                        ->    Metric Filters  ->

# LAB: Automate the alerting and remediation of an IAM user creation event

IAM User Creation -> CloudTrail -> CloudWatch-Logs -> Metric Filters -> SNS Topic

Step - 1 Create an SNS topic and register an email address for that topic.

Step - 2 Create a CloudTrail trail for the IAM user creation API call and then connect to CloudWatch-Logs so we can receive a periodic stream of API events.

Step - 3 Create an S3 bucket within the CloudTrail interface where the logs will be stored.

Step - 4 Now we must integrate the CloudTrail into the CloudWatch-Logs. Go to the CloudTrail interface, then select configure CloudWatch. Create a new or select an existing log group.

Step - 5 Again within the CloudTrail interface, we move to the next screen which create an IAM role which will allow CloudTrail to develiver events to CloudWatch-Logs.

Step - 6 If everything goes well, after a few seconds you will see a condfigured CloudWatch log group and a corresponding IAM role again within the same CloudTrail interface.

Step - 7 Verify the linking of CloudTrail to CloudWatch Logs by doing CloudWatch -> Logs and you will see the corresponding log group that you configured. If you open that log group, you will find at least one stream which we configured for each region. If you open that stream, you will start seeing the events.

Method - I
----------

Step - 8A The first method that we will use will comprise of metrics. So we go the log group and then select filters within that entry as shown below:

 ![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/selectmetric.png) 
 
Step - 9A Create the filter pattern that needs to be detected:

            { $.eventName = "CreateUser" }

then select assign filter.

Step - 10A Now we must name the filter i.e. we name our filter as shown below:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/namefilter.png) 
 
Step 11A Now we create an alarm from that metric filter. So we select the alarm option and then select the SNS topic that we had previously created.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/alarmcreated.png)

So now we go ahead and create an IAM user and the following is the chain of events:

a) An IAM user is created so IAM calls the create user API
b) This API call is logged by CloudTrail
c) CloudTrail sends this API log to the log group that we configured in CloudWatch-Logs
d) That particular log group within CloudWatch-Logs has a metric filter configured
e) The metric filter on that log group then does a pattern search for the create user API 
f) When the metric filter finds the call, it generates an alarm which is linked to the SNS topic that we created
g) That alarm has our user email registered

The only problem here is that if you go ahead and create a user, you will not get an email immediately and that is because the API call and its visibility is not real-time if you use the architecture just described i.e. using metric filters and alarms. Moreover, even the information that you received is not very detailed, just very basic info. So now we go ahead and use an alternate architecture.

                CloudWatch Metric Filters               vs.                 CloudWatch Events
            
            i) Not real-time, slow e.g. in minutes                            Real-time
            
            ii) Information in email very basic                             More detailed information
            

Method - II
-----------

Step - 8B Go to CloudWatch and select Rules in the left menu.

Step - 9B Select "Event Pattern" within the CloudWatch rules.

Step - 10B Now we create a CloudWach rule. We select "Service Name" as IAM, then "Event Type" as AWS API call via CloudTrail then we select that we want this rule to be triggered on a specific operation i.e. "CreateUser" and review that the event pattern preview has been correspondingly updated. Finally, select the SNS topic that we created last time.

Now if we go ahead and create an IAM user, we will get the notification almost instantly, and ite email contains a lot of detailed information in JSON format. A pro tip is to copy this JSON and paste it into an JSON editor for better reading.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/createrule.png)

# Automating the Response Side (In addition to the Detection and Alerting Side that we did before)

The core of response is ths delivered by the CloudWatch Events which is at the heart of this automation. It is the HUB of automating alerting.

        The "Event Pattern" of CloudWatch Events is the heart of automating alerting as it can trigger a lot of stuff e.g. 
        it can trigger a lambda function.
        
        CloudWatch Events (Event Pattern) -> Lambda
                                             System Manager (patching)
                                      * Imp: SNS (Email, ticket generation or integration with 3rd party softwares)
                                             SQS
                                             
So the amazing idea is that if we have some advance knowledge of some security incident that can occur, then we can create a CloudWatch Event rule which monitors an occurrence of that incident and triggers an automated response to remediate or mitigate the security repercussions.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudwatchevent.png)

Now the cool thing is that, we now have the power to automate the detection, alerting and response to a LOT of different security incidents. Two examples are quoted below:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/examplesofautomation.png)


