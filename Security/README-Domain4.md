# Identity and Access Management

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/centralizedlogging.png)

IAM Policies
------------
An IAM policy document always starts with a version and then has statement(s). A statement consists of the following:

- Effect: "Allow" OR "Deny"
- Resource: "*" OR "arn" or ["arn1", "arn2", "arn3"...]
- Action: "*" OR "service:operation"
- Condition: CONDITION

Principal is the identity to which the policy document is applied to i.e. the one which is going to access this resource.
