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
