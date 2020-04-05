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
