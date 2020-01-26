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
      Step 4 - Very important, you can set the rule to allow, block or simply count.
      
      ** EXAM: We have the options of allow, block or count and count is amazing as it allows us to test a rule 
      in live environment without actually blocking anything to better study the potential impact.
      
      Conditions applied to -> Rules  applied to -> ACL applied to -> cloudFront Distribution.
      
      ** EXAM TIP: WAF CANNOT block Denial of Service or Distributed Denial of Service attacks since WAF is looking at 
      suspicious stuff inside the traffic not the traffic as a whole i.e. DDoS. For that, we need to use AWS Shield.
      
      
AWS Shield comes in two flovors:

1. AWS Shield Basic, free included with WAF, provides basic DDoS protection.
2. AWS Shield Advanced, paid, provides more comprehensive protection, 24/7 response team, protects additional resources such as ELB, protects against more attacks e.g. SYN flooding, HTTP flooding, monetary protection, $3000/month/oerganization.
 

# VPC Design and Security

Probably, one of the most important parts of AWS. Basic unit of construction.

            * VPCs can be connected to private on-premise networks using VPNs and direct connect.
            
    Customer (public Internet) -> public zone (S3, dynamoDB, IGW, Lambda) -> VPC -(VPNs, Direct C.)--> On-premise networks 

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/vpc.png)

Step - 1: Create a VPC (CIDR: 10.0.1.0/24) with no inbound/outbound stuff, nothing, just empty private region inside AWS.
Step - 2: To give public access to this VPC, we need to create and attach an IGW to the VPC router.
Step - 3: What this does is attach it to the VPC router which is a logical entity that you do not get exposure to, but you can control this virtual object using route tables.

Private subnets: No security exposure because it is entirely private and not accessible fronm the Internet.

Step - 4: Create subnets, called db1: 10.0.1.0/28 and this allows for 16 hosts in this one based on the VPC CIDR 10.0.1.0/24
Step - 5: Create another db subnet db2: 10.0.1.16/28 to make sure it has non-overlapping ip range with the first db subnet
Step - 6: Create an app subnet: 10.0.1.32/28 
Step - 7: Create an app subnet: 10.0.1.48/28

Step 8 - Now create web subnets: 10.0.1.64/28
Step 9 - Now create 2nd subne:   10.0.1.80/28

            2 web subnets: public
            2 app subnets: private
            2 db subnets:  private
            
            ** Exam: Technically, being a public subnet just means that you have a subnet which has a route attached to it 
            as 0.0.0.0/0 Route and the destination of that route is the IGW. To achieve that,  you need to create a public 
            route table for this VPC, associate this route table with the two public web subnets and create a route 
            0.0.0.0/0 and the last step required is to attach a public or elastic IP to your resources.
            
Step 10 - Create a new rotue table and associate it with the public subnets (web subnets)
Step 11 - Add a default route 0.0.0.0/0 to this route table
Step 12 - Last step to provide public access to web subnets is to attach a public or elastic IP to the EC2s in web subnets.

            ** Architecture point: According to this guy, by correctly using AWS Security Groups and ACLs, you can still
            have a secure network even without a multi-tiered architecture. BUT, if you need different routing decisions for 
            different subnets, then you DO need the different tiers. But overall, its still a good idea to create multiple
            tiers.
            
# Security Groups

Its a group that applies to a group of EC2 resources by allowing or blocking traffic.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/securitygroups.png)
         
            * Security groups are created inside a VPC, so a security group inside a VPC cannot be assigned resources to 
            another resource in another VPC. However, a security group in one VPC CAN refer to another security group in 
            another VPC.
            
            Exam: Security groups are not associated with EC2 instances, basically they are attached or associated with 
            network interfaces on that EC2 instance. So a fun thing could be that you could be super-granular and allow 
            traffic to one particular interface and not allow traffic to another interface. Interestingly, the same is the 
            case with RDS instances, they put netwotk interfaces in the VPC, and it is to those NICs that security groups
            attach to.
            
            ** Exam: Security groups are stateful.
            
            * Exam: You can reference another security group as the source or destination of traffic which essentially means
            that any network interfaces that have this other security group attached are allowed inbound/outbound traffic.
            
            * A powerful feature is "SELF-REFERENCE" i.e. in the edit rules, you can reference the security group itself
            which would allow us to create a security group between multiple network interfaces and define traffic within
            that group of network interfaces. It allows you for grouping of EC2 instances which can talk to each other.
            
            * Exam: Security groups can reference each other in other VPC as long as they are within the same region. They
            CANNOT reference other security groups in other regions.
            
            * Important: Security groups filter traffic just before it hits network interfaces, but theoretically the 
            traffic is already inside the AWS and VPC.
            
            Inbound traffic: Internet -> IGW -> NACLs -> Security Groups -> Network Interfaces -> EC2
            
            Outbound traffic: EC2 -> NACLs -> Security Groups -> IGW -> Internet
            
# Network ACLs

Network Access Control Lists are slightly different from security groups.

NACLs are always associated with subnets and are always processed at subnet boundaries.

            *Exam: One NACL can be associated with multiple subnets, but one subnet is only associated with single NACL.
            *Exam: NACLs are stateless.
            *Exam: Rules are processed in priority order.
            *Exam: NACLs are always processed before security groups when leaving or entering subnets.
            *Exam: Any traffic within the subnet is not affected by NACLs since traffic is not exiting or entering subnets
            *Exam: We CANNOT attach NACLs directly to logical names of resources, but can attach them to subnets, or you can 
            attach them to resources using their IP addresses or CIDR ranges

NACLs are stateless, so for example in the diagram below i.e. communication between two instances in different subnets, then NACLs will be triggered 4 times actually:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/nacl.png)
