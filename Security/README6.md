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


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrstandard.png)
2. Other Account: The only difference from the standard access is that in this case, the destination account needs to add a bucket policy to the destination bucket so that the replication engine can replicate/write files to the dest. bucket. Npte, this is in addition to the IAM role.
* Exam tip: Putting objects in the destinaton in another account, but remember the source owner will remain the owner even for the objects in that bucket. This however, causes some serious configuration problems, so one option is that you could edit the settings on replication and change the replication configuration to change the owenership to the dest. account i.e. any objects being sent to the dest will become dest account's ownership.


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrownerchange.png)
3. Owner Change: You need to change the replication configuration file. Simply change the setting that allows for the destination account to become owner once the objects have been replicated in this bucket and AWS will automatically update hte replication configuration file automatically. This resolves a lot of configuration issues which would otherwise occur if you were to put the objects in the destination while retainign ownership.


![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crrkms.png)
4. KMS: Basically, you encrypt the replicated objects so that the destination gets encrypted objects (SSE-KMS). For that, you need to add KMS permission to the IAM role i.e. allow the replication engine to access the KMS encryption keys which will be used to encrypt the objects being copied.

