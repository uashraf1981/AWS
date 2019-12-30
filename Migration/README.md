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
