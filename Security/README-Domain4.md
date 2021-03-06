# Identity and Access Management

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/centralizedlogging.png)

IAM Policies
------------
An IAM policy document always starts with a version and then has statement(s). A statement consists of the following:

- Effect: "Allow" OR "Deny". -> every policy has an implicit deny so if nothing here, then implicit DENY. Deny takes default.
- Resource: "*" OR "arn" or ["arn1", "arn2", "arn3"...] * applies to ALL AWS resources or some e.g. ALL S3 resources.
                             generally the format is arn:partition:service:region:account:resource-type:resource
- Action: "*" OR "service:operation" and is completely related to the resource defined above. So if you defined the resource  
                  as the S3 bucket in the resource above, then the action applies to that resource e.g.
                  S3: GetObjct
                  S3: Get
                  IAM: Change Password
                  IAM: Access Key
- Condition: CONDITION.  Is used to apply additional resrtrictions e.g. based on IP addresses, allow only during certain times, allow based on username for example, or allow access only to a certain folder in S3 bucket, or force that MFA be used.

Principal is the identity to who the policy document is applied to i.e. the one which is going to access this resource. If principal is applied to a resource like S3, then we need the principal otherwise if it is applied to an identity then we don't need to specify the principal as it is assumed that it applies to the identity.

* If you don't specify a principal and the pricipal was supposed to be a resource, then you will get an error.

* Remember that Resource and Action work in pair e.g. you may allow * i.e. access to all resources (generally not a good idea) but then you can filter based on the action 

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/policy.png)

The policy above says that user Jack can lise the la-folder if he puts the strong /Jack/ at the end of the bucket name when trying to list the contents so he must do that when trying to do that or will be denied.

Same technique required for the getobject and putobject operations.

* Learned an interesting thing that you can go to the terminal of an instance and create profiles for users by giving their usernames and passwords/keys. This comes in handy when you want to access a resource let's say S3 as that particular user so you can simply refer to that profile.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/terminal.png)

# Users, Groups and Roles

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/usersgroupsandroles.png)

User
----
A user is a true identity in IAM and that is because every user has an ARN which is user-friendly to read since it has the username at the end. When you create an IAM user, they have NO permissions associated with them.

* Access keys, usernames and passwords are an example of long-term credentials.
Access Key = Access Key ID (public) + Secret access key
* You can have 2 access keys per IAM user. You can activate them, deactivate them and even delete them. In key rotation, you generate a new access key and delete the old one.

* Exam Tip: If a pre-signed URL is based on an IAM user key, then inactivating or deleting that key will make the pre-signed URL immediately inactive.

* Users in an AWS account = 5000
* Roles in an AWS account = 1000
* Groups in an AWS account = 300
* Customer managed policies = 1500

* Exam Tip: You must use the access keys to login to the command line. You cannot user username and password to login to the commandline or the API. Username and password are ONLY for console access.

There is a tab in the IAM menu called "Access Advisor" which lets us know what permissions have been assigned to this user and when was the last time that this permission was used.

Credential Report
-----------------
Lists details of all the users in the account including ARN, date of creation, whether password enabled, last time the password was used, whether access key 1, whether MFA is enabled etc.

* Security Tip: Always use the minimum number of IAM users required. So if some accounts are not being used etc, then disable them as a security measure. Access key leakage is one of the most common security problem in cloud, and when you hear in the news about some compromise, usually it involves the leakage of the key.

Groups
------
Groups act as a container for users. 
A group can have many IAM users and an IAM user can have many groups.
Groups are not real identities, so they cannot call AWS services directly, and don't have long term credentials, they don't have usernames and they CAN'T be reference in a policy. You can have a policy assigned TO a group but not the other way around. Groups are simply a collection of users.

Roles
-----
You never login to a role i.e. you cannot have it a starting point. They don't have usernames or passwords. You basically ASSUME a role.

Trust Policy
-------------
A trust policy is the entity that checks whether a thing is allowed to assume the role that they are trying to.

Permission Policy
-----------------
The permission policy is the list of things that the role can itself do.

SSM role is an instance role which allows an EC2 instance to interact with the Systems Manager.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/trustpolicy.png)

Above is the trust policy for the SSM role.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/policyforrole.png)

* Another big use of role is account switching within AWS console. The role name is OrganizationalAccessRole.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/accountrole.png)

* Roles are mainly how AWS services access other services. Same for EC2 instance, when you apply role to an instance, then that instance gets the permissions to call different services.

* Roles are also used to delegate access to other accounts which you trust. Basically you create that role and then define a trust policy which allows those remote accounts to assume that role.

* You can also define roles to cater for break-glass situations when an admin may need to access a role.

TO ASSUME A ROLE: You need to call STS:AssumeRole API call.

* Exam Tip: Once you have assumed a role, you can also assume another role.

Revoke Sessions:
----------------
* Exam Tip: A very cool way to revoke access to active roles without impacting the role itself. You just kick the active entity using that role instantly. 

        The revoke role session works by changing the trust policy dynamically by adding a new condition that all access
        before current time should be denied, so all active connections are revoked whereas new sessions would still be 
        allowed.
        
* Exam tip: Remember, you can never LOGIN to roles.

# Permission Boundaries and Policy Evaluation

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/permissionsboundaries.png)

Permissions boundary is a set of access that an entity (user, role etc) should NEVER access. It acts as a safety net for adherence to organizational policies.

* A permissions boundary does not grant permissions, it only restricts permissions. A permissions boundary can further restrict the permissions granted by the permission policy, so it can be a subset of the permissions policy.

It is basically a safety net and is defined just like a policy. It says that this identity can at max have access to these things in AWS. Then in the day to day operations, you assign permissions etc but the permissions boundary will always be enforced and prevent any access assigned beyond its permissions.

* Exam Tip: Having a permission in the permissions boundary does not mean that we are actually assigning permission. It just means that if allowed through the permission policy, then the permission boundary allows it. Remember, it's just a boundary, it doesn't actually grant those permissions.

* Permission boundaries can be applied to any identity in AWS that has permission policies and that includes IAM users, IAM roles. Permission boundaries limit as to what a child account can do, and that includes the root account as well.

* One good use case is if you want an IAM user who can assign permissions to other users or create/delete other users. What permissions boundaries allow is that you can create an organizational bundary policy e.g. what organizations users can do, so we can apply that policy to everyone, but then we can give our account administrator power over these but then apply the boundary policy to keep check and balance in place and that includes that admin himself as well e.g. they should not be able to remove those boundaries. Thus, you can delegate permissions to other admins to create/manage users without worrying about excessive access.

Policy Evaluation
-----------------
Organizations boundaries -> User or role boundaries -> Role policies -> Permission 

# Organizarions and Service Control Policies

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/organdservicecontrol.png)

AWS Organizations
-----------------
AWS organizations is a multi-account management system. Companies were running hundreds of accounts and managing them for billing and security separately. AWS organizations is the solution for this problem and allows for managing the security and billing in a hierarchical manner.

The account from which you create the AWS organization becomes the Master account.
* The Master account is a special acccount and is the only account that can't be restricted.

* When you create an organization, you can either select just consolidated billing or "All features" which includes:
  - Service control policy 
  - Cross acccount roles
  - Role switching between accounts

* Basically we are creating inverted trees.

Organizational Unit (OU)
-----------------------
Is basically a container for other accounts or other OUs as well. Applying policies to an OU will affect all accounts within that OU as well as all child OUs or accounts.

* Exam Tip: Whenever an account is created or joins an organization, it is immediately impacted by the service control policy either by the parent OU or all the way up to the root. If you invite an account to your organization, you need to do a handshake protocol and the owner of this account needs to accept joining your organization.

Service Control Policy
----------------------
Are basically permission control boundaries. They can be attach to:
 - Accounts
 - OUs (Organizational Units)
 - root objects
 
              The Master Acccount is the ONLY account in AWS which CANNOT be restricted by Service Control Policy. Usually
              Service Control Policy is very powerful and applies to everything inlcluding root users, but does not apply 
              to the master account. But you cannot restrict the root user in the master account with ser. control policy.
              
              Generally a master account is used to role shift into other accounts e.g. when we do account switching.
              
              If you want to insist on applying security control policy restrictions then do all your policies in child
              accounts, but not the master account.
              
              Even root is blocked by service control policy but the master policy is not impacted in any case.

# Resource Policies: S3 Bucket Policies

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/s3resourcepolicy.png)

* Just like any other policy, it is a JSON document which contains one or more resources. The main difference is that identity policies are associated with identities (think IAM) whereas these are assigned to resources.

Identity Policies: Can be assigned to users, groups or roles.
Resource Policies: Are assigned to resources and identify what identities can access them and which permissions they have.

* Exam Tip: Remember that identity policies are applied to identities within the account whereas the resource policies are applied even to resource outside the account e.g. untuehtnicated users accessing S3 buckets.

* Exam Tip: For resource policies, you MUST specify the "Principal" to which this policy applies. and this is the real power i.e. they influence any identities accessing them.

* Exam Question: If you want to define permissions for anonymous users e.g. Internet users on a resource such as an S3 bucket, then you should use a resource poicy not an identity policy.

          Examples of restrictions you can enforce on S3:
          
           - Access only from specific IP addresses
           - Access from a specific VPC
           - Allow only HTTP referral i.e. people visiting my website can access images stored on S3 for that website e.g.
           - Allow only MFA authenticated users to access this bucket
           - Allow other resources to access the bucket e.g. inventory
           
Since S3 have identity policies and resource policies therefore, the end result is a merge of these policies when someone or some resource is accessing that bucket. Remember, deny takes precedence.
           
# KMS Key Policies

One good use case if you want to SEPARATE the day to day management of encryption keys from the actualy encryption/decryption of data.

* When you create a key, by default, the key policy allows the root user to administrer keys, but if you remove that access
you can actually LOCK YOURSELF OUT of the key policy and need to contact AWS support.

Key administrators
------------------
Are allowed via key policy to administer CMK (Customer Master Key) and do operations like create, delete, update, enable etc
but remember, they CANNOT perform ACTUAL cryptographic operations.


Generally, KMS basically generates CMK (Customer Master Keys). The CMK is not actually used to encrypt data itself as they can only encrypt small amounts of data. Instead, they are used to generate DATA KEYS which are then used to encrypt massive amounts of data.

CMK -> generates two keys: i) paintext data key which is used to actually encrupt and then discarded and 
                           ii) data key encrypted by CMK and then stored with the encrypted data itself
                           
The next time you need to decrypt the data, you first decrypt the data key using CMK and then use the decrypted key to decrypt the actual data so CMK basically encrypts/decrypts data keys.

* Exam tip: If you want to allow cross-account access to keys then you need to edit the key policy and you need to do that carefully. Basically we give access to CMK so that the cross-account can interact with the CMK.


# Cross-Account Access to S3 Buckets and Objects

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/s3crossaccountaccess.png)

ACLs and S3 Permissions
-----------------------
ACLs are a legacy access control mechanism. 

* Exam Tip: If Account B user PUTs an object in our S3 bucket, then even though the object is in OUR bucket, Account B user is still the owner of that object. So if you have activated cross-region replication on your S3, then this object will NOT be replicated since you don't have permiossons on it.

Bucket Policies and S3 Permissions
----------------------------------
* Exam Tip: The solution to the above problem is Bucket policies. Bucket policies allow for advanced conditions and do not even need to involve IAM. This is handled within the policy document which forces that no one is allowed to put objects into our bucket unless they also allow us to become owners of those objects.

But still, the user in Account B needs to decide if he is willing to allow permissions or not.

IAM Role and S3 Bucket
----------------------
User in Account B assume a role of a user in Account A so when they put an object in the bucket, they are basically putting objects as a user of Account A so Account A in this case will have complete access to the objects and this is the crucial difference from ACLs and Bucket Permissions.

# Identity Federation

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/identityfederation1.png)

* Exam Tip: Since there is a limit on the number of IAM users that you can create and there is an overhead on maintaining the IAM yourself, therefore, it is a good idea especially if you have a large number of users to use identity federation.

Steps:
1. Login to AWS application
2. App redirects you to 3rd party authentication such as gmail which generates an ID token (very large encoded piece of information) if login successful. Remember, this token expires after a certain period.
3. App can now swap that ID token with a Cognito token. Cognito is interesting because it allows for several things such as identity merging in which the same user can use different identities such as gmail or facebook
4. You can then use the Cognito token and swap it for an AWS role (STS) i.e temporary credentials. There is a trust policy which allows us to assume this role. If successful in assuming this role, we will get the temporary access credentials.
5. Acess AWS using STS credentials


SAML Based Federation
---------------------
More of an enterprise level solution. The first main difference is that you USE YOUR OWN IDENTITY PROVIDERand not famous third parties like gmail or fb. 

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/saml.png)

* Exam Tip: You also need to use your own identity portal. You basically go to the identity portal which then authenticates through the internal database.

1. Contact your IDP portal
2. After authenication, the portal returns a SAML assertion which redirects you to a single (URL) of AWS Single Sign On
3. Now you go to the AWS SSO and give it the SAML assertion (a kind of token as explained before)
4. The AWS SSO validates this SAML assertiomn with the STS
5. STS generates a new token and a new URL which is sent to AWS SSO and AWS SSO sends back to us
6. From now on,you can use the assumed role and go to the URL

For CLI access, we skip the step of AWS SSO and once we get the credentials from IDP portal, we go diretly to STS and get the credentials i.e. assume the role.



Web Identity Federation -> Mostly used for mobile apps with large number of users who already have 3rd party accounts
SAML 2.0 -> Mostly used in enterprise environments when you already got an internal identity provider e.g. Active Directory

# AWS Systems Manager Parameter Store

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/sysmanagerparameterstore.png)

The basic idea is that if you are managing hundreds or thousands of EC2 instances from which some instances need to be restarted, shut down, replaced etc. all the time, then you need a place to store the configurations including passwords, connection strings.

Parameter store is a stateless, serverless storage entity that can store both data + secrets. So it avoids the security problem of secret leakage by virtue of them getting committed to git repositories.

                1. You can pull all or any specific piece of information
                2. Data is stored hierarchically
                3. Data is versioned and access is logged and can be audited
                
                Serverless, resilient and scalable.
                

You can choose to use KMS keys to encrypt your parameters.


Once you have stored your parameters in the lambda store, you can then go ahead and create a lambda function which access these different parameters.

# Troubleshoot Permissions Union

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootpermissions.png)

* Persmission union is a situation when an identity accesses a resource and multiple permissions apply such as IAM, User and Access Control List.

Generally, the priority is as follows: explicity deny -> explicity allow -> implicit deny

# Troubleshooting Cross-Account Roles

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootcrossaccount.png)

1. The first step in troubleshooting cross-account roles, is to determine if you can actually assume the role or not.
2. The second step is if you CAN assume the role, but you don't have the permissions.
3. You chould have cloudtrail enabled and if you have done so, then you can lookup all events related to "AssumeRole" which will tell you what went wrong when someone tried to assume role.
4. Next step is the trust policy of the role, including trust relationships and check which principals are allowed to assume this role.
5. Check trust relationships---- they are basically concerned with delegating trust to other accounts so they can assume roles.
6. Check the identity on both sides whether it is the exact identity that has been given the permissions. This is especially complicate because an IAM user can assume another role and then that role can assume another role and so on.

* Exam Tip: One possible problem is the scenario shown below:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/crossaccountproblem.png)

The account on the left has a trust relationship with the middle one and the middle one can assume roles in that account. However, the rightmost account may also have a trust relationship with the middle account and by virtue of that, can assume role in the left most account even though that account didn't explicitly give permission for this. This is called the "Confused Deputy Problem"

Solution: Require External ID, which is like an extra piece of information and which requires that sts:ExternalID to be specified which is basically the original AWS account ID from which this access started.... so even if we have done assume role then assume role then assume role, then the external ID would allow us to check where did this identity originate from and block based on that. This is done within the trust's role policy so it is used while assuming role. It is like an MFA.

A trust policy basically defines, who can assume a role.

# Troubleshooting Identity Federation

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootidentity.png)

# Troubleshoot KMS Key Policies

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/troubleshootkms.png)

* KMS is designed to separate key management people from key users. For instance, admins who manage keys cannot use them for encryption/decryption whereas those who use the keys cannot manage them for instance. If we remove the part from the policy then we will be left with an unusable key.

* Customer Master Keys (CMKs) are generally not used to encrypt data, they are used to encrypt date encryption keys.

* KMS limits: 5500 decrypts per second and 10,000 encrypts per second.

There are two roles in terms of KMS:

1. Key Admins: They enable, disable, schedule key deletion, rotate keys
2. Key Users: They use the keys.

These two roles are KEPT COMPLETELY SEPARATE through permissions.
