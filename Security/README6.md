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
