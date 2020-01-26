# Web Application Firewall and AWS Shield

AWS WAF is essentially a layer 7 firewall i.e. application level firewall.

Sits in front of the CloudFront or ALB and intercepts traffic before it hits these services.

Customers --> WAF --> CloudFront or Application Load Balancers (ALBs)

It examines not only the traffic meta-data i.e. headers, but also looks inside the payload to detect known exploits like SQL injection, Cross-Site Scripting etc.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/awswaf.png)

Web ACLs: The basic component of the WAF are the Web ACLs (Access Control Lists). Each WAF is made up of rules, and each rule has conditions. Some rules have multiple conditions.

In this example, we will use WAF to block traffic from a particular IP address (condition 1) AND if it has SQL injection code in it (condition 2) i.e. two conditions must be met. Note that SSH traffic won't be blocked even though its coming from the same range.

      Web ACL = Rules + Conditions
      
      e.g. Condition = Block traffic from Australia then we use this in a rule which may also contain other conditions.
      
      Two type of Rules:
      
      1. Regular Rules: Which match traffic e.g. any traffic containing SQL injection code needs to be blocked.
      2. Rate based Rules: Which count the number of breaches of a particular condition in a window of time e.g. if we observe 
      strange traffic from this ip adress three times e.g. then a rate-based rule is triggered.
      
      ** EXAM: When you are creating the ACL, you can create a blacklist type ACL or whitelist type ACL.
      
      ** EXAM: Applying the WAF is done through the resource itself e.g. go to CloudFront then select this WAF or 
      technically the ACL inside the WAF.
      
      ** EXAM: Rules always follow AND for conditions i.e. condition 1 AND condition 2.
      
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/wafrule.png)


      Create an ACL:
      
      Step 1 - create some conditions e.g ip is coming from australia
      Step 2 - create rule and add this condition plus other conditions if you want
      Step 3 - create a new ACL abd add rules to it.
 
 
 
