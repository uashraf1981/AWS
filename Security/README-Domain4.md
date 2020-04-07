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
