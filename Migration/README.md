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



# Coursera Course on Cloud Migration Using AWS

Always remember Why a company is moving to the cloud comes before
            Phases:
                1. Migration Preparation and Business Planning
                2. Portfolio Discovery
                3. Design, migrate and validate applications
                4. Operate
                
Remember, its a walk in the park to migrate isolated, stand-alone systems, but for the systems which are interconnected, migrating these interconnections is the real challenge.

AWS Application Discovery Service --> Helps analyze our current environment by gathering statistics so that we can make coherent decision about whether which set of applications can bs migrated directlty, which need to be retired, etc.

* Each application must be designed, migrated and validated based on its specific components and requirements i.e. especially in heterogeneous environments, you cannot treat the whole system as one and individual applications must be catered to.

            The major services are:

            1. AWS Server Migration Service (SMS)
            2. AWS Database Migration Service (DMS)
            3. CloudEndure Migration
            
In the operate phase, we look at the ongoing operations and we look at when can we turn off the old systems.

AWS Cloud Adoption Framework (CAF)
----------------------------------

             Strategies for Cloud Migration include 6 R's:
             
             1. Rehost     -> Migrate quickly (Lift and shift)
             2. Replatform -> Migrate but optimize in the process (Lift, tinker and shift) tinker=e.g. move to RDS
             3. Repurchase -> Upgrade to a different license model or product while in migration e.g comm to enterp. license
             4. Refactor   -> Rearchitect the system to add new features, scale or performance, expeneive but best results
             5. Retire     -> Legacy systems or assets that can be retired that are no longer in use
             6. Retain.    -> Retain things as is... sometimes you are not ready, or business feels more comfortable

Week 2
-------
One of the major benefits of cloud migration is scalability.

Scaling Constraints
-------------------
Not everything can be easily scaled. For example, if scaling requires additional licenses, then you may need to purchase them and thus this can make scaling a little challenging e.g. if you need to purchase licenses, wait for approvals or other business processes. So the following could be some interesting work-arounds to solve these problems:
               
               1. Use managed caches and queues
               2. Offloading unncessary work from unscalable servers
               3. Life-cyle stale data off the overloaded server to make space/capacity
               
               * What to do about session data e.g. our applications want to scale but if we do horizontal scaling, then we  
               may loose the session data, so the solution for that is to off-load the session data from the instances i.e.     
               store the session data somewhere else and this would enable sort of loose coulping.

Horizontal Scaling: Add similar instances or resources, can scale up and down dynamically.
Vertical Scaling: Make the original resource more powerful, but the downside is there is some downtime and you can't scale down back again.

Considerations with Migrating DBs vs. Applications
--------------------------------------------------
Always a good idea to de-couple your databases from your applications and other components. The major difference being that with applications, the switch happens almost instantly i.e. its binary, either its on premise or on the cloud. However, with data, there are tansition periods when it is being accessed from on-premise and then from cloud.

               Checklist before staring migration:
               
               - Find out things like database tables, schema, other limitations. 
               - Find out if your applications can addord downtime, and if so, how much. 
               - Find out how much of your data you need access to now or in the immediate future. 
               - Find out about your network e.g. how much bandwidth can it support.
               - Since you may be placing additional load on your DB during migration, find out how much can you put on it
               - Personnel aspects, tranining and knowledge required e.g. network requirements, firewalls, database etc.
               - Time planning is also important. Since refactoring is also usually involved for apps, dbs etc. so a lot of 
               effort is required so be realistic when estimating the timelines.

AWS Server Migration Service (SMS)
----------------------------------
Allows you to easily migrate thousands of servers. Makes it easy to schedule, track migrations. Also allows for tracking incremental changes to live server volumes and applying to the migrated servers. 

Currently it ONLY supports migrations for the following:

1. VMWare vSphere
2. Miscrosoft Hyper-V
3. Azure virtual machines

Server VMs ------replicated as----> Amazon Machine Images (AMIs) on the cloud-----launch----> as EC2 instances

You need to use the AWS connector which is basically a BSD VM virtual machine which you need to install on your premises and which will facilitate in migration. Allows up to 50 concurrent machines.

Monitoring of your migrations is made possible by using Amazon CloudWatch.

VM Import/Export Service
------------------------
An amazing service which allows you to easily import your vmware machines from your premises to the AWS environment.

AWS Application Discovery Service
---------------------------------
This service allows two modes of operation:

1. Agent less Discovery: works only with VMWare VMs with AWS agent-less discovery virtual appliance, called discovery connector and provides general info. What type and size of EC2 instances to use, peak and average CPU, RAM usage. But you cannot peek into the VMs for process info or network connections

2. Agent based Discovery: We install AWS Application Discovery agent on each of the virtual machines and physical servers. Will provide detailed process info, inbound and outbound network connections and other detailed statistics.

Lab Basics
----------
Scenario, migrate an on-premise ghost application to AWS. There are 2 boxes:

 - Application Server running Ghost
 - Database server running MySQL

You are told that both are running Ubuntu (probably).

            
         
