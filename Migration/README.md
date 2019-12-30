# Cloud Migration

AWS Migration Hub
-----------------
Provides a dashboard and a central point for managing a migration project. 
There are two mai components:

1. Discovery: Uses tools from AWS to audit existing servers and applications i.e. get an idea of existing infrastructure. This is done by agents installed (but can also be agentless) on these servers which send this info to the service discovery repository which is then displayed on to the migration hub.
2. Migration: Uses tools from AWS to migrate a project to the AWS cloud platform

Note: Migration CAN be done without the discovery phase.

Migration Hub Strategies
------------------------
There are two main types of migration hub strategies:

1. Discover and Migrate - Suits complex environments in which we have heterogeneous servers, different operating system, applications and several unknowns. The benefit of this approach is that you can normalize your environment BEFORE you actually migrate, thus making the whole process simpler and minimizing business interruptions later on.

The Discovery phase has several sub-phases inside: 

    i) Choose and deploy AWS discovery tools 
    
    ii) View discovered servers
    
    iii) Group servers as applications
    
 The Migration phase has several sub-phases inside: 
 
    i) Connect migration tools to migration hub
    
    ii) Migrate using the migration tools
    
 Tracking:
 
    i) Track status of migrations
    
    
2. Migrate Directly - Suites environments which are homogeneous e.g. standard VMware images and hypervisors and in which we have already a good understanding of the environment.

VMWare Cloud on AWS
-------------------
In contrast to other compute services on AWS like EC2, VMWare on AWS runs on the bare metal hardware i.e. it is not multi-tenanted and there is not XEN hypervisor, so it is not shared.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/xen.png)![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/VMWare.png)

Even if you use a dedicated EC2 instance, the difference would still be that even that dedicated EC2 instance would be using the Xen hypervisor but in case of VMWare, it would be doing that without any sort of hypervisor. VMWare also uses a hypervisor ESXi hypervisor, but it is different from the Xen hypervisor.

Migration Strategies
--------------------
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/migration.png)


Roles and Persmissions (Authentication) for AWS Migration Hub
---------------------------------------------------------
AWS Migration Hub requires roles and persmnissions (Authentication) for performing migration tasks. You need to do two things:

1. First you need to define a policy so that AWS Data Migration Service (DMS) can access any service as shown by wildcard:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/policy.png)

The wildcard shows that the 

2. You need to define a trust policy for AWS Data Migration Service (DMS) to assume that role:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/trustpolicy.png)

In summary we ALWAYS have [AWS Migration Hub Policy] --> [Trust Policy]

The list of services required by AWS Migration Hub include:

a) AWSMigrationHubDiscoveryAccess -> Access to call Application Discovery Service
b) AWSMigrationHubFullAccess -> Full access to console/cli.
c) AWSMigrationHubSMSAccess -> To allow hub to receive notifications from server migration tool.
d) AWSMigrationHubDMSAccess -> To allow hub to receive notifications from Data Migration Service tool.

Miscellaneous:
-------------
AWS Migration Hub supports services in several regions of the world. 

AWS Migration Hub - Discovery Services:
---------------------------------------
It uses either Discovery Agents or Discovery Connector:
i) Discovery Agent -> Allows agent installation (on both virtual and physical services), het env., slow but more detailed.  
                      Remember, these agents require outbound access from on-premise to AWS, so firewall needs exceptions.
ii) Discovery Connector -> Used for VMWare based machines, does NOT require agent installation, useful for known env, fast.

Installing AWS Discovery Agent:
-------------------------------

For this, you need to first insall Microsoft VC++ runtime library on the server and then windows-based discovery agent. You need to login to the server as administrator and run from the command line the windows agent installer. Remember, you may want to disable the auto-update feature by using that flag in the command line to avoid updates during the migration process.

AWS Migration Hub Tools:
-----------------------
The AWS Migration Hub has several tools from AWS (e.g. AWS Server Migration Service) and several from its partners. The biggest tool is perhaps AWS Server Migration Service (SMS) which allows you to migrate:

VMWare-Sphere/Microsoft-Hyper-V   -->  AMI Images ready to be deployed onto EC2 instances.

An amazing feature of the Server Migration Service is that it tracks incremental changes on your on-premise servers and only updates the AMI images in the AWS ecosystem based on only the changes actually tracked i.e. syncs both.

        Steps followed by SMS:
        
            i)    Schedule migration task
            ii)   Take snapshots and upload VMDK to S3
            iii)  Convert VMDK to EBS snapshot
            iv)   Create Amazon Machine Image (AMI)

