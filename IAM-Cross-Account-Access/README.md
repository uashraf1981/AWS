# AWS Cross-Account Access
Background
----------
IAM is the Identity and Access Management Service of AWS and offers four major modes of access:

1. Security Groups
2. Policies
3. Roles 
4. SAML

Security Groups
---------------
Security Groups are used to pool users together which belong to a certain logical class e.g. admins, developers etc. 

Policies
--------
Policies define the administrative access priviliges for accessing different resources. AWS has two types of policies:

1. Identity based policies (policies that you attach to IAM roles, groups and users) e.g. John user is allowed to get 
data from a particular table in DynamoDB or is allowed to RunInstance for a particular EC2 instance you list the resources 
and specify the permissions that they have. 

2. Resource based policies are attached to AWS resources e.g. S3 buckets, SQS, AWS Key Management System (KMS). 
For resource-based policies, basically you specify who can ACCESS this resource e.g. if it is an S3 bucket, then you 
specify WHO can access this bucket etc. Used a lot of the time in conjunction with Roles for cross-account access scenarios.

We can create policies I three ways:

1.	Copy and existing AWS policy and then customize it
2.	Use policy generator
3.	Create your own from scratch

Roles
-----
Roles are an interesting feature of the AWS IAM. Basically, e.g. if you want EC2 to access another resource in AWS, you
need to store access credentials (passwords or secret access key) on the EC2 instance which is a very bad idea. Another
situation is when you want to give cross-account access to some users, then in both these situations, you can use a "Role".
The easiest way to understand a role is that it is basically like a "Hat" which you can wear i.e. it is some form of
temporary credentials which allow temporary access. At the backend, the technology that makes this possible is the 
Security Token Service (STS). The STS service is used for the roles which are basically temporary credentials to access different services. The way this works is by making API calls to the endpoint sts.amazonaws.com which is the global end point for STS or you can configure local ones as well. Roles are necessary because we cannot attach policies to EC2 instances e.g. to allow them access to S3. 

Roles are used for:
1.	Access to services by EC2
2.	Delegation i.e. allowing user from another AWS account to access
3.	Federation i.e. allow integration with existing accounts e.g. gmail


Integrating on-premise IAM system (Windows Active Directory) with AWS IAM
-------------------------------------------------------------------------
Basically, if you want to integrate an existing on premise IAM system e.g. Microsoft Active Directory with AWS, then you have two options:	

1.	Use SAML and build trust relationships
2.	Use AWS Active Directory for Single-Sign On by building trust relationships

Security Assertion Markup Language (SAML)
The security assertion markup language is used a lot in hybrid environments when you want Amazon to work with other identity systems, such as Microsoft Active Directory. We don’t have to create accounts again on AWS, they can simply use the existing on-premise solution.

AWS Directory Service is another way to integrate.


How Does Cross Account Access Work in AWS
-----------------------------------------

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/IAM-Cross-Account-Access/Delegation.png)

1. In the Development Account, create a role CrossAccountSignin and specify cross-account access and give the ID of the development acct. The same wizard allows you to attach a policy and two options are usually popular i.e. PowerUserAccess and AdministratorAccess. The power users has all access except for IAM. When done, note the Amazon Resource Name (ARN).

2. Login as Admin, and modify the policy for relevant users and enable access to STS:AssumeRole. Specify the ARN of CrossAccountSignin as the resource for the action part of the policy. 

3. Another important step is that when you call the STS:AssumeRole API, it gets temporary credentials for cross-account access, but it is MUCH SUPERIOR to write a script to automatically convert this into a console sign-in to the remote account. To do that, you can create a script that takes advantage of a feature in AWS known as the federation endpoint (https://signin.aws.amazon.com/federation). You can make a request to this endpoint and pass it temporary security credentials that you get from AssumeRole. The endpoint returns a sign-in token that you can then use to construct a console URL. This console URL lets a user sign in to the console without having to supply a username and password, because the URL contains a token that indicates that the user is already authenticated.

