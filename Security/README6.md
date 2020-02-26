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
