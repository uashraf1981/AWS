# Centralized Logging

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/centralizedlogging.png)

From a blast radius perspective, each account has separate users, groups and roles. Users in one account cannot access other accounts e.g. an IAM user in developer cannot access an IAM user account in production for example. Good design from a blast radius perspective so ideally, development account, staging account and production account. If you need to, you can role-shift into other accounts.

* So it is nice from a security perspective that you have segregated these accounts, but it also presents a challenege in the sense that you would have setup CloudTrails in each of these account separately which logs to a bucket in each of these accounts.

Solution: Use centralized logging by configuring your cloud trails aggregated in a central bucket for easier and better visibility. We will also ensure that these logs are encrypted using KMS.

Demo Steps:
 
Step 1 - Go to CloudTrail and create SecurityTrailMaster trail in the master account. Select apply trail to all regions. Enable data events for S3 and Lambda functions in this account.

Step 2 - From within the CloudTrail screen, create a bucket called secprod-multiaccount-logs.

Step 3 - Still from within the same environment, go to advanced and enable KMS encryption and create a new key. 

At this point, you will start to see logs in the CloudWatch logs i.e. the master account side of things is working fine.

Step 4 - Go to the bucket and grab bucket's ARN.

Step 5 - Make sure this bucket has appropriate permissions so that CloudTrail from different accounts can write to this bucket.

Note down the account numbers of production and development accounts.

Step 6 - Go to the bucket permissions, we need to modify the second block of code i.e. add the new account numbers to be allowed permission as only the current policy only allows the master account to write to this bucket as shown below.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/bucketrpermissionpolicy.png)

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/policyupdated.png)

Step 7 - Go the production account and create a new trail (sectrailprod), apply for all regions, for all events, all S3 and lambda data events. We won't be creating a new S3 bucket. Important, now HERE you mention the multiaccount bucket as dest.

Step 8 - Repeat the process for the staging account.

Now if you go to the multiaccount bucket, you will see folders for each account.

* If you see the logs delivered by these accounts to the master account, you will see that they are only utilizing AES-256 encryption i.e. server side encryption because we did not specify KMS encryption to be used. Part of the reason that even though have a KMS key in the master account, but these sub-account do not have access to the KMS key.

Mofidying Persmission on the KMS key

Step 9 - Go to IAM -> Encryption Keys -> Select the KMS key that we configured. Grab the ARN of this key.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/updatedkeypolicy.png)

Step 10 - Now we are going to update the key policy as shown above and add the two other accounts in permissions.

Step 11 - Now we login to each of the other accounts and update the CloudTrail to tell it to utilize KMS and the specific key that was condfigured in the master account by specifying the ARN of the master KMS key as shown below.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/updating.png)

* What the last steps mean is that the other accounts are securely delivering logs to the mmultiaccount bucket using KMS encryption and you can check this by going to the multi-account bucket and checking the server-side encryption of the patticular logs delivered by those account trails. It means that only those people who have access to the KMS key in the master account have the ability to decrypt these logs so basically

  Set of people who can encrypt the log files != Set of people who can decrypt those log files
  
* Addtionally, you can also disable deletes and only allow writes to ensure integrity of data.

** This strategy of logging into an audit account which accumulates logs from a fleet of AWS accounts across the organization and is useful for stringent auditing purposes.

** Exam: Using KMS encryption is great to ensure separation of permission to allow only a certain set of people to encrypt and only a small set of people to decrypt.

# Troubleshooting Logging
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshooting.png)

* CloudTrail logging is enabled by default and there will always be some logs available for you to see in the recent events of CloudTrail. However, you cannot customize these logs and moreover, they are only available for 90 days.

* Keep in mind that for CloudTrail, any kind of serious trouble will likely be because you don't have a trail configured or configured incorrectly. It would never be from the Event History tabs, but always in the Trail tab of CloudTrail.

* Limit = 5 trails per region. If you have one trail for all regions, then that will count towards that 5 limit.

* Global Service Events: Are evens that have a global end-point and two of them are IAM And STS (Secure Token Service). These events are only delivered to the trail if the trail is configured to accept them.

* Exam question: When you create a trail from the GUI (not from the command line), then they WILL INCLUDE global service events and if you have multiple trails created from the GUI, then you may get duplicate events as all will include their respective copies of global service events.
SOLUTION: Have a single trail which caters to these global service events and do not include global service events in your other tails and for this you will need to use the CLI as it is a little difficult to do from the GUI.

* EXAM TIP: If you are not receiving S3 or lambda events in your trail, then remember this is an easy fix. You just need to enable them in the trail settings since by default trails only include management events and not data events (object level activities in S3 or lambda)

* A lot of the times, the reason why logs are not being delivered is because the bucket policy is not properly configured.

* Every trail in CloudTrail is set to be delivered to a particular Log Group in CloudWatch. A cool trick to detect of any loggin problem is by within this screen, check the last log delivered time and if is greater than an hour, then there is a problem which you need to investigate.

* First place to check is the IAM role that CloudTrail is assuming to push logs to CloudWatch, so you need to make sure that the IAM role and attached policy is correct. Generally, you need the policy to create logstream and put events.

* Last step is to go to CloudWatch and check that the logs are being properly delivered,

FLOW LOGS
---------
* Flow logs are also delivered to CloudWatch.
* You can go to VPC -> Flow Logs to see the VPC level flow logs that are configured to be delivered to CloudWatch and also lists the IAM role used (also includes the ARN of that role).

      * Remember the concept of "Capture Points" for VPC flow logs i.e. you can set for flow logs to be captured at the   
      following levels:
      
      1. Capture point for flow logs at VPC level or
      2. Capture point for flow logs at subnet level or
      3. Capture point for flow logs at network interface level
      
      * Remember, these level get inherited e.g. if you apply at VPC level (#1), then they ALSO get applied to #2 and #3
      * Exam TIP: A lot of the times you will get a question that these flow logs are applied at the wrong level e.g. at the 
      network interface level or the subnet level when you really want to log traffic between different subnets.
      
      * EXAM TIP: You may not be using the correct filter e.g. you can apply flow log to capture:
      
      1. ALL traffic
      2. Accept traffic or
      3. Reject traffic
      
      although no plausible reason why we wouldn't just simply use the all traffic option.

* PING from outside -> Security Group (stateful) -> NACL -> EC2
            EC2     -> NACL -> Security Group (stateful) -> outside
            
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/permissions.png)

In the above example, the second ping is not being allowed outbound because of a NACL which is denying since it is stateless, the security group is not blocking it.

Route 53 DNS Logs
-----------------
* Just like others, it has its own role and policies attached and if you open the log group for Route 53 DNS Logs, you will see logstreams for each individual edge location (named after the closest international airport).

* Remember, you can point the nameservers to an external provider, but in that case, you will NOT be able to use Route 53 query logging, so they need to be Route 53 name servers in order to be logged. 

* EXAM tip: Query logging only works with "public hosted zones" they don't work with private hosted zones i.e. within VPC.

EC2 Logs
--------
* CloudWatch agent must be installed onto EC2
* Appropriate permissions must be provided
* Configuration is also sometimes a problem. USe System manager and a parameter store to automatically push the configs to the agent. You can use the run command to perform actions on scale.

S3 Access Logs
--------------
* S3 access logs are a bit of an outlier in the sense that they don't integrate with CloudWatch logs. You do configuration on a per bucket basis and get the logs delivered to a bucket. Remember that you need to assign permission to the LogDelivery/Group from within the bucket interface. Any changes may take up to an hour, it is NOT real-time. 
* Exam tip: Log delivery related questions but the main idea is that log delivery in S3 is NOT real-time.

# Multi-Account: Troubleshoot Logging

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootmultiaccount.png)

One of the challenges for multi-account logging and logging in general is that there are no error messages just lack of logs. For example, if cloudtrail from multiple accounts is pushing logs into a bucket in an audit account and if it doesn't have appropriate permissions, you will not get an error message, but there simply won't be any logs when you open the bucket.

* The way permissions work is that you add permissions to the bucket to allow CloudTrails from other accounts to be able to write into specific folders within the bucket and you do that by modifying the policy. The folder name added is usually the account id.

        Step - 1: Check resource permissions i.e. bucket policy that CloudTrails have appropriate permissions.
        Step - 2: Check the KMS key that you defined to be used by all the reporting accounts, so go to the key in the 
        master account, check the key policy to check what operations are allowed and by whom.
        
The way that you setup CloudWatch in an account to push data to S3 in another account is by going to CloudWatch in the sub-account, go to the logs group, and then right click to export the data to S3 into a separate account, but you DO sitll need the ability to write to the destination bucket.

* Never assume that the destination bucket name being given to you is accurate, always double check since bucket permissions get changed sometimes, sometimes when people make that bucket again, they may make a typo or there may be a malicious activity which changed the bucket name etc.

      * Exam: The only way to get real-time logging in multi-account setting is by streaming the log group to lambda which 
      can then push the log data in real-time into the appropriate bucket. Kinesis can also get the same job done !
      
      * EXAM tip: Provide least privilege access e.g. CloudTrail writing to a bucket in a master account should only have 
      write access, not read access and similarly, only a select group of people need to have read access to that bucket 
      in the master account and should not have write access i.e. try to minimize privilege.
      
      * You should always use log file protection.
      
* Worth exploring: What is the difference between exporting from CloudTrail to master account vs. CloudWatch to master acct.


# S3 Forced Encryption

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/centralizedlogging.png)

      * Exam Tip: There will be questions about how to ensure that any objects uploaded to S3 are encrypted or any specifi
      type of encryption is done whenever an object is uploaded to S3. It is important to realize that buckets NEVER get
      encrypted. It is the OBJECTS that are uploaded into the bucket that are encrypted. This can be achieved by having a
      bucket policy that forces that any object being put in the bucket should be encrypted. This is achieved by denying an 
      action of putting object in S3 if the put request header doesn't contain the encrypted header.
      
      BUT remember, this was the only functionality available. Now we have another option in bucket policy options which is
      the default encryption option i.e. if an encryption is not specified for an uploaded object and we have selected 
      default encryption, then the S3 will by default encrypt (usinf AES-256 or AWS-KMS) the objects. 
      
      * Exam Tip: Remember that even if you have turned on default encryption e.g. AES-256, but have a policy on denying
      uploads for objects which do not have the encryption header, then it will be denied since the policy applies first! so
      if you insist on it then it will be denied.
      
      To include encryption header when uploading an object:
         aws s3 cp ./picture.jpg s3://bucket-name --sse AES256
      otherwise if you do:
         aws s3 cp ./picture.jpg s3://bucket-name
      then this will be denied even if you have default encryption turned on.
      
      Historically, the bucket policy to force encryption was used, but now mostly people just use default encryption.
      
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/denys3.png)      

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/bucketdefault.png)


# S3: Cross Region Replication (CRR) Security

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrsecurity.png)

Basically, a bucket in region A replicates objects into a bucket in another region. 
* Remember, it only replicates objects starting from the time that replication was turned on, will not replicate existing i.e. no retroactive replication.
* Only replicates unencrypted objects or standard encrypted objects i.e. SSE-S3
    - Unencrypted objects -> supported
    - SSE-S3  -> standard encrypted objects supported
    - SSE-C -> customer encrypted objects NOT supported
    - SSE-KMS supported but needs a lot of configuration

* By default, object ownerships and ACLs are also ditto replicated, but of course you could change it.
* The same storage class is also used e.g. frequent access type will be again frequent access
* Exam Tip: Lifecyle events ARE NOT replicated, only human or application based actions are replicated
* Exam Tip: If I allow you to put objects in my bucket, then I can give you permission on that, but since I would still not
have permission on those objects, therefore, those objects will not be replicated.

Types of CRR:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrstandard.png)

1. Standard (Both buckets in same account): Use IAM role which has permission to access objects in bucket A and put those in bucket B.


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrotheraccount.png)
2. Other Account: The only difference from the standard access is that in this case, the destination account needs to add a bucket policy to the destination bucket so that the replication engine can replicate/write files to the dest. bucket. Npte, this is in addition to the IAM role.
* Exam tip: Putting objects in the destinaton in another account, but remember the source owner will remain the owner even for the objects in that bucket. This however, causes some serious configuration problems, so one option is that you could edit the settings on replication and change the replication configuration to change the owenership to the dest. account i.e. any objects being sent to the dest will become dest account's ownership.


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrownerchange.png)
3. Owner Change: You need to change the replication configuration file. Simply change the setting that allows for the destination account to become owner once the objects have been replicated in this bucket and AWS will automatically update hte replication configuration file automatically. This resolves a lot of configuration issues which would otherwise occur if you were to put the objects in the destination while retainign ownership.


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrkms.png)
4. KMS: Basically, you encrypt the replicated objects so that the destination gets encrypted objects (SSE-KMS). For that, you need to add KMS permission to the IAM role i.e. allow the replication engine to access the KMS encryption keys which will be used to encrypt the objects being copied.


# Web Application Firewall and WAF Shield

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/waf.png)

WAF is a layer 7 firewall.

It sits in front of the CloudFront or Application Load Balancers and is hit before any of these.

The base entity is a Web Access Control List (ACL).

Rules are of two types:

1. Normal rules e.g. traffic from this IP containg SQL injection should be blocked
2. Rate-based rule - if some condition is violated a certain number of times during a window of time e.g. if we see some strange traffic coming from this IP 3 times in an hour then rate-based rule can be triggered.

Web ACLs --> contains Rules --> contain conditions

* Remember, you can associate the firewall to an ALB or CloudFront distribution.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/wafdetails.png)

You can create a blacklist style ACL or whitelist style ACL.

* Interestingly, you can apply geo-restriction directly in the CloudFront distribution or you can apply an ACL. However, it takes nearly an hour whereas with ACL providing geo-restriction, you get results very quickly.

* There is an option of either to allow, deny or count individual rules within an ACL. A smart idea is to first try the count thing as it allows you to observe how many times was your rule triggered instead of actually going ahead with the live website. 


* WAF is a great product in the sense that it applies restrictions before the traffic reaches your edge infrastructure.

* AWS Shield is used to protect against DoS and DDoS attacks.

AWS Shield comes in two flavors:

1. AWS Shield Standard - comes free with the WAF
2. AWS Shield Advanced - $3000/month - expands protection to ELBs, CloudFront distributions, 24/7 DDoS Response Team, insurance included against bill spike due to DDoS attack and also protects against many advanced types of attacks.

WAF                                  vs.                             Shield
---------------------------------------------------------------------------------------
1. Fiter traffic for known attacks                   Protection against network attacks (DDoS)
2. Define flexible rules                             Real-time response time team
3. Rate Limit rules                                  Protection against bill usage due to spikes due to DDoS
4. Firewall outside of edge services


# VPC Design and Security

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/vpcdesignandsecurity.png)

Note the AWS public space zone and the on-premise network. There is no connection between on-premise network and VPC and you need to create direct connect or VPN connections to this VPC.

Note: By default, the VPC boundary is non-permeable i.e. unless you explicitly allow it, traffic can't get in or out.

VPC limits the blast radius from a networking perspective, but not useful if the account gets hacked.

* To provide Internet connectivity for public users or connectivity to the end points (lambda). You can do that by creating an Internet gateway IGW. Inside every VPC is a VPC router which is a logical construct, which you cannot actually access. However, you can control it by using Route Tables.

Pivate Subnets: They don't have any route to the public internet OR to the AWS public end points such as lambda etc.

* Public subnet: Basically just means that you have am IGW associated with the VPC and a default route 0.0.0.0/0 pointing to the IGW so that any traffic from that subnet is routed by default to the gateway. You do that by:

 1. Creating a public Route Table
 2. Attach this route table to the subnets you want to make public
 3. Add local routes e.g. 10.x.x.x. and the DEFAULT ROUTE: 0.0.0.0./0 -> IGW
 4. Assigning a public or Elastic IP to your resources such as EC2 instances in the subnet
 and you are good to go.

** AT this point, the security has changed suddenly as your machines are publicly accessible.

* Exam: It IS possible to NOT utilize this multi-tier architecture i.e. web tier, app tier and database tier by using different subnets. Now, AWS offers the capability to achieve that by using things like Network ACLs etc. and that is actually a good idea because the less things we have to manage, the less prone to errors it is and the less security mistakes we make. Always err on the side of minimal artchitecture footprint.

# Security Groups

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/securitygroups.png)

* Security groups are created within a VPC. So a security group that is created inside a VPC cannot be assigned to resources inside another VPC. Be careful, security groups CAN refer to other security groups inside other VPCs, but you can't assign  a security group within one VPC to resources in another security group.

* Another important point is that security groups are never technically associated with EC2 instances, tehcnically speaking, they are associated with a network interface and in turn that network interface is attached with an instance. Since an EC2 instance can have multiple interfaces therefore you can have different security groups on different interfaces of an EC2 instance with different rules.

* Security groups are stateful.
* Security groups have an implicit deny default rule.
* You cannot explicitly deny traffic, you can explicitly allow certain types of traffic and use the implicit deny.
* An interesting point about security groups is that in the rule configuration tab, I can refer to logical resources as being the source e.g. I can refer to other security groups as being the source for inbound rules. This means that we are allowing incoming traffic from that security group. Technically that means that any network interface with the referred security group attached will be allowed to send the approved type of inbound traffic to us.

* Cool: You can also specify the source as yourself i.e. in the security group rule settings, you can specify the source for the approved inbound traffic as yourself. What you can achieve with this is that you can have a group of Network Interfaces on a group of EC2 instances and allow to create a sort of group in which everyone can send traffic to each other.

* Several other resources such as RDS instances also utilize security groups not just EC2 instances.

* Exam Tip: An important point to note is that security groups can reference other security groups or other logical resources directly if they are in the same region. If they are in another region, then you need to use CIDR or IP ranges.

Traffic coming from Internet --> NACLs ---> Security Groups ---> EC2 instances
Traffic coming from EC2     ---> Security Groups ---> NACLs ---> Internet

# Network ACLs

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/nacl.png)

* NACLs are associated with subnets, they CANNOT be directly attached to resources such as EC2. They can be assocaited with multiple subnets but one subnet can only be associated with ONE NACL.

* Unlike security groups, NACLs process rules in order.
* In security groups, we did not have explicit deny, but in NACLs we do have explicit deny.
* Remember, rules are always processed in order and as soon as the first rule matches, no further rules down are checked.
* NACLs only check traffic at the subnet boundaries, NOT the traffic within the subnet. For that, use security groups.
* A bi-driectional communication between two EC2 instances in two different subnets will hit NACLs 4 times since its stateless
* Unlike Security Groups, NACLs CANNOT reference logical resources, only CIDR ranges and IP addresses.
* Unlike security groups in which you can only explicitly allow, in NACL you can explicitly allow and explicitly deny.
* Exam tip: At the end of the inbound list of rules, there is an implicity deny rule which you cannot delete. You need to add rules on top of it.
* By default, Network ACLs allow all incoming and outgoing traffic.

# VPC Peering

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/vpcpeering.png)

* VPC peers are non-transitive.
* VPC peering CANNOT be done with VPCs which have overlapping CIDR ranges.
* Please note that just sending & accepting peering connection is not enough. The VPC router does not have any route to the peered VPC and we need to add a route with the CIDR range of the other VPC in this router before actual routing can occur. Similarly, symmetrically, you need to configure a similar reverse route in the route table of the other VPC.
* One last thing that you need to do is to update the security group inbound rules in an instance in one VPC and reference the security group in the other peered VPC and allow some inbound traffic from that security group to this EC2 i.e. Security group peering. As SGs basically are attached to NICs so only traffic from those specific machines will be allowed to come.

* This security group peering trick will ONLY work if the two VPCs that are to be peered are in the same region.

* For VPC peering between VPCs in different regions, we repeat the same steps i.e. send/accept invitation, add routes to both the routing tables at the two ends. 

* Exam Tip: When VPC peers are in different regions, regardless of whether both belong to the same account, you CANNOT reference the Security Groups in the other VPC. You MUST define inbound rules etc in Security Groups based on CIDR range.

Case 1: Peering VPCs in same account and same region --> Can reference Security groups
Case 2: Peering VPCs in same/diff account and different regions --> Cannot reference Security groups
Case 3: Peering VPCs in diff accounts but in same region --> Can reference Security groups but need accountid+SG_number

So basically SAME REGION == Can Reference SGs regardless of same account or diff account
             DIFF REGION == Cannot Reference SGs regardless of same account or diff account
             
* Exam Tip: When adding Routes to route tables for enabling VPC peering, you can add complete CIDR range of the other VPC or speccific individual IP addresses.

* Exam Tip: When you allow Security Group in one VPC access from another Security Group in a different VPC (peering VPC), then you basically delegate you security to the owner of that SG and if he attaches his SG to hundreds of machines, you will have no control over the security of your instances attached to this SG which allowed the other SG access. So, perhaps it could be interesting to use NACls which allow explicitly denys.
             
# VPC Endpoints

VPC allow you to access AWS services like S3 and DynamoDB without an IGW attached to your VPC as this improves security. Traditionally, the IGW was required in any case. They come in two flavors:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/gatewayendpoints.png)

1. Gateway Endpoints:- You need to create a rule in the VPC router which routes traffic towards that destination service. Usually this entry is in the form of a prefix list (pl-1234abde, vpcaw19212) where pl part is the prefix list and represents logically the S3 e.g. and vpcaw19212 represents the vpc end-point.

* DevSecOps: One very interesting application of gateway endpoints is that we can attach policies to it (but can't to interface endpoints). One usecase is we attach a policy for our EC2 instances to access code repositories in S3 buckets without allowing full access to all AWS resources. This makes our VPC kind of private with just enough access to get code updates from the repo. Another way to look at it is that we can have a policy on the bucket such that denies all access except for requests coming from the gateway endpoint i.e. from the vpc.

* Exam Tip: You cannot provide security/protect the gateway end-point using NACLs or Security Groups as they DON'T operate on gateway endpoints, BUT you CAN use gateway end-point policies or policies on the resources itself e.g. S3 policies.

* Exam Tip: End-points don't work across regions i.e. the endpoint and the resource must be in the same region.
* Exam Tip: Endpoints only use IPv4 addresses.
* Exam Tip: You cannot use DNS or VPN to access a gateway endpoint.
* Exam Tip: You can't utilize the source IP condition in policies when using VPC endpoint. Instead use VPC id or number. Otherwise your policies would be broken.

2. Interface Endpoints

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/interfaceendpoints.png)

* The difference from gateway endpoints is that gateway endpoint is virtual but interface endpoint actually injects a real interface in your VPC.

* Another diff is that gateway interfaces live within a region and have high availability, but interface endpoints live within a specific availability zone and if that availability zone fails then they are dead. If you want high availability for interface endpoints, then inject them in each availabilty zone separately so that some failover is available.

** Exam Tip Major: Interface endpoints allow private connetion between your VPC machines and AWS resources such as S3 but more importantly, *** they allow private connections from 3rd parties e.g. your on-premise network to the VPC ***

* They are a powerful feature which allow you to have private VPCs which you can access from on-premise and which can connect to public AWS services like buckets again privately.

* Another major difference is that you can attach Security Groups to the interface endpoint because Security Groups are always attachable to interface endpoints and even more cool is that you can reference other Security Groups logically and allow traffic from other private VPCs for example.

* They are much better and powerful than gateway endpoints, but the only downside compared to gateway endpoints is that they cannot have policies attached directly to them like gateway endpoints.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/interfaceendpoint2.png)

When you add an interface endpoint then you get a vpc reference which is region wide and which you can use to access the VPC. However, if you want to access the specific location where the endpoint was added, then you can use DNS entries.

Usage: Simply modify your code running on EC2 instances and tell it to use the vpcendpoint reference and then you will send your data to that endpoints which will then directly send it to the AWS resources such as the S3 instead of routing it through the VPC router and the internet gateway.

Regional DNS Address: Region wide vpc reference for the endpoint. Needs internet gateway to be accessed
Zonal DNS Address: Refers specifically to the exact interface endpoint in that exact AZ.

* When creating the interface endpoint, you are allowed to create a private DNS which will then override the public DNS that is created by default.

# Serverless Security

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/serverlesssecurity.png)

Lambda is a product that runs functions.

* A lambda function policy is a typical resource policy just like we attach to S3 and it controls who can run that function. Remember that the root user in the account to which this lambda function belongs is given the permission by default to run it.

* Services can also run lambda function, but usually it is done automatically e.g. if S3 is configured to run this function then it will be automatically given the permission, you don't need to manually do that.

* When you want to grant other accounts permission to run the lambda function, then yo need to do that manually by passing it the ID of the other account.  

* IAM execution role is about giving the lambda function the permission to invoke other services such as S3 via temporarty credentials. This needs to be done manually again e.g. lambda function may generate some data when invoked and needs to store logs into cloudwatch etc. then it needs an IAM execution policy to store them.

* Exam Tip: For all PULL, PUSH or POLL based functionality, lambda needs IAM execution policy configured for permission
* Exam Tip: For EVENT-DRIVEN invocation, lambda does not need IAM execution policy.


So Function Policy --> Who can invoke Lambda
   Execution Policy --> Determines which services can lambda call e.g. store data in cloudwatch
   
# NAT Gateways

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/natgateways.png)

NAT instances or better, NAT gateways are public facing devices and are typically used to provide Internet connectivity to instances residing in subnets. Since they are public facing, they need Elastic IP addresses.

NAT gateways reside in SINGLE subnet and use ELASTIC IPs.

NAT gateways cannot use Security groups, the only option to secure them is through NACLs an that too by attaching the NACLs to the subnet that the NAT GW resides in.

NAT gateways are not highly available i.e. they don't failover between different subnets.

NAT Gateways are a managed service so you cannot SSH into the NAT GW instance. 

* Exam Tip: You cannot setup port forwarding on NAT GW (you could do that previously with NAT instance)

Summary:
- You cannot use Security Group on the NAT GW
- You cannot do port forwarding on NAT GW
- You cannot SSH into NAT GW
- You CAN only control security through NACL on the subnet

New NAT gateways (NAT GW) can only be secured by NACLs and not Security Groups as they don't recognize Security Groups.


# Egress Only Internet Gateways

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/egressonlyinternetgateways.png)

All EC2 instances in AWS have private IPv4 addresses attached to them. The gateway handles the Internet connectivity. If you want to give an instance a public IP address, then AWS achieves this illusion through the Internet gateway. Basically what it does is that on the way out, it swaps the private ip address with a public ip address and when the traffic is coming into the AWS, it does the reverse swap.

NAT GWs also allow private EC2 instances to have Internet access.

* Exam Tip: With IPv6 if you create a VPC, then every instance will have a PUBLIC IP address in contrast to IPv4 address. That is, IPV4 addresses are by default private while IPv6 addresses are by default publicly routable and that poses a security challenge.

Egress Only GW:

- Works only with IPv6
- It allows egress only, ofcourse it is stateful so allows response back in, but connection must be initiated from inside.

The main contribution is that egress only GW only allows outbound initiated traffic compared to GW which allows inbound initiated also. Remember, egress only is still stateful and will definitely allow the response back traffic.

* You cannot use SEcurity Groups on the gateway itself. You CAN use NACLs on the subnets in which your EC2 instances reside.

# Bastion Hosts / Jumo Boxes

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/bastionhosts.png)

Imagine you have some instances in a private subnet which have sensitive data. You can use bastion host/jump box which is basically just an instance with a public IP address and you open ONE port on that e.g. 22 for SSH so that administrations and technical staff can connect through that to this instance and then "jump" to the sensitive private instances.

* Main point is that it is super locked down. You will have a single public subnet with a single bastion host. The bastion host connects to the Internet GW to access the Internet.

* Exam Tip: This powerful security is provided by a single security group attach to this bastion host which only has a SINGLE allow rule for that allowed port and only from a known set of IP addresses. Additionally, since SG don't allow explicit blocks (we already leveraged SG to only allow access to a specific port from specific ip addresses) we use NACLs to block all other kinds of accesses.

We can attach roles to this bastion host so that it can monitor logs and send to cloudwatch.

* Exam Tip: You can get a question that VPN or Direct Connect is not an option so how to connect to private instances. Then you can say bastion host which also allows federated identities. Since we don't have VPN or Direct Connect with On-premise network, so we can't use federated identities since it would require synchronization. Over the public Internet its not that safe. So bastion hosts does not need a direct connection with on-premise network and allows for the on-premise users to use the public Internet to access the bastion host and then access the private instances. YOU typically also use IDS/IPS on bastion hosts and then use the role to ingest logs into cloudwatch so you have a nice hardended public end point.


Since it offers a single entry point so easier to manage.

# Troubleshooting a VPC

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootingvpc.png)

Routing
-------
You can use routing as a security feature as no route means traffic restricted. Remember that when peering VPCs, you can either give the complete CIDRS block ranges (e.g. /24) or better yet, give specific IP addresses (/32).

Overlapping CIDR ranges in peered VPCs are bad so you will not be able to create peers.

NACLS
-----
- Only tool that can allow explicit blocks so if you are attacked e.g. SSH brute force, use NACL blocks
- Are attached to subnets
- Stateless, so you need rules in both directions
- Processing order is important
- NACLs only apply to traffic that cross the subnet boundaries, so it will not work b/w two instances within same subnet
- You can only use IP addresses or CIDR blocks for rules, cannot use reference names of SGs etc
- One subnet = one NACL
- One NACL can be applied to multiple subnets but reverse is not true

SGs
---
- They apply to network interfaces, they don't apply to subnets or even directly to vpcs or instances
- Can only allow traffic, they CANNOT deny traffic explicitly (only implicit deny is there i.e. you don't allow)
- Reference other logical resources such as themselves or other security groups
- Allowing traffic from itself means you can create a group of network interfaces
- They can reference logical resources only within the same region, for different regions, they lose this ability so use IPs
- One SG can be connected to multiple interfaces, one interface can have multiple SGs
- You cannot connect SG to end point

Logging and Monitoring
----------------------

Always check VPC Flow logs for allow or deny messages. You may have NACL and Security Groups having different configs e.g. SG is allowing, while NACL is blocking or vice versa

Use Cloudwatch logs/metrics


- 



