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
