# AWS Host/Hypervisor Security

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/hostandhypervisorsecurity.png)

EC2 instances in AWS run on shared resources but are isolated in the sense that every traffic must pass through the hypervisor, then the virutal interface, then the seccurity group, then the firewall and then the physical interface.

MEMORY
------
* Exam: When you terminate an instance, the portion of memory that it was using is srubbed i.e. zeroed by the hypervisor BEFORE returning it to the pool.

DISKS
-----
* Exam: For disks, the contents are non-volatile. The contents are zeroed out JUST BEFORE it is shared with the instance. Note this is in contrast to the memory. So its dangerous, because the actual data is still on the diskin the pool and is only scrubbed immediately before giving it to an instance.

If you have compliance requirements, you need to manually zero out or wipe data the contents before terminating the instance.
You can use automated tools which search for volumes that are not attached and scrub them.

Always encrypt your volumes using kms.

# Host Proxy Servers
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/hostproxyservers.png)

Basically you install proxy software on an instance and that is called a host proxy server.

Typical deployment: intances --> Proxy --> NAT --> Public Internet

We use host proxy server. Filetering in AWS occurs in security group level or NACL. SGs work on interface and NACLs work on traffic which traverses subnets. But SG or NACLs only work on IP, CIDR or protocol. IF you want to put checks on authentication mechanisms etc. then you need to use proxy. Basically you can ask instances in the lower subnet to authenticate to our proxy server using any custom software and then traffic will proceed further so we are inserting custom authentication in our AWS ecosystem.

Another good use case can be that the proxy uses identity federation with an external authentication source. So the proxy server can allow/deny access to the Internet based on:

- username
- password
- session state e.g. allow only if certain applicatio is logged on or not
- only allow/deny to certain DNS names

# Host Based IDS/IPS

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/hostbasedIDS.png)

Host based IDS works at instance level and checks file and memory contents to detect malicious activity. Host based IDS usually run on-box.

AWS Security Products:

1. WAF: Prevents threats even before it arrives at our edge.

2. IDS Appliance: Off box solution, monitors data as it moves into the platform i.e. on internal network and services. It can take actions such as block traffic or notify admins.

3. AWS Cconfig: Ensures stable configuration across all acccounts.

4. AWS Inspector: Monitors resources and looks for known exploits or quationable OS/software. Software patches monitoring.

5. Host based IDS: Handles everything else. 

Typically what you do is you use Systems Manager to install IDS on multiple instances and then the IDS injects any logs that it creates into CloudWatch logs which then trigger lambda based actions or alerts.

IDS observes things such as system or OS files, memory usage or leaks or other behavior of applications, privilege escalation etc. to identify malware infections.

While many AWS products complement an IDS, but you definitely need a dedicaated IDS for complete implementation.

# AWS Systems Manager

It is a systems management tool mostly used in the operational side of things. System Manager's functionality can be divided into three areas:

1. Insights:- Gather information
2. Actions:- Perform actions
3. Shared tooling

Insights
--------

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/systemsmanagerinsights.png)

Insights offers: 

a) Inventory and 
b) Compliance

and both are supported by the SSM "State Manager" component.

Systems manager agent is installed by default on a lot of instances by default. Every instance which has it installed is called a "managed instance" so technically even on-premise servers with the agent installed are managed instances.

You need to give those instance a role which allows them to interact with the SSM manager so that they can perform the SSM< actions using the locally installed agent. By gigving the role. the agent regularly remotely send information such as network configurations, AWS products, CPU cores, speeds, roles, etc from the instancce to the SSM.

Patch Baseline:- A powerful concept in which we attach a patch basline to SSM which tells it that basically we want these patches to be installed on all our instances at all times and THIS helps in ensuring compliance of the data i.e. it can provide a compliance or non-compliance report about each resource (instance).

* Exam tip: SSM is a regional service i.e. it is enabled on a per-region basis. But what you CAN do is to enable all these regional SSMs to write to an S3 bucket and that bucket data can then by used in Athena or Quicksight to provide a global information status of your ecosystem.

Actions
-------

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/systemsmanageractions.png)

Workflow or automation: is defined as a series of actions which can be as simple as rebooting instancs or installing a bespoke application or run scripts. The persmission is achieved through roles and the interesting thing is that it gives way more firepower to the executing entity. For example, junior admins can be given broad permissions.

Run Commands -> Allow us to run commands at scale on managed instances. The commands are called "documents" and the result is typically a configuration change on that instance.

Patch Manahger -> Allows you to install patches on instances to perform remedial work.

Shared Tooling
--------------

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/systemsmanagersharedtooling.png)

Parameter Store: Parameter store stores is an AWS service that stores configuration data and secrets and its hierarchical so you can add permissions at levels of trees. You can also use KMS to encrypt secrets.

Personal Health Dashboard
-------------------------

Provides a personalized health board which provides global information about the instances.

# Packet Capture on EC2

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/packetcapture.png)

We cannot sniff remote traffic, we can only install sniffers onto an EC2 instance which will then capture traffic coming into and going out of the instance. Even if you set the sniffer in promiscuous mode, you will still only see the traffic for and from the instance.

Scenarios:

- Want to see deeper into VPC traffic between components
- Support IDS/IPS
- Help in debug
- Assist in checking performance of other components

Remember,it works in complement with VPC. VPC flow logs only provide meta data of traffic, but they do not provide actual packets so in that case you need a sniffer application. You can install sniffer on lots of instances and then tell them to send their logs to cloudwatch logs.

TCPDump is one of the most popular packet sniffer used in AWS.

* Exam Tip: Only the sniffer allows to capture real world traffic.

