# Cloud Migration

AWS Migration Hub
-----------------
Provides a dashboard and a central point for managing a migration project. 
There are two main components:

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

# LAB 1

Lab Basics
----------
Scenario, migrate an on-premise ghost application to AWS. There are 2 boxes:

 - Application Server running Ghost
 - Database server running MySQL

You are told that both are running Ubuntu (probably).

On-premise network simulated as being in region Us-West-2 --> and goal is to shift it to --> AWS US-East-1).

Step-1 Install AWS Application Discovery Agent:
-----------------------------------------------
We will first poke around the two boxes (VMs) and go ahead and install AWS discovery agents on both machines. The AWS Applicaton Discovery Agent is supposed to capture several things including configuration settings, performance, processes running and network connections on the end points that it is installed.      

They are using Cloud9 to allow us access to AWS console and SSH access into the two boxes. 

Step 1.1: Create a new user using IAM called "migration-user".
Step 1.2: Give him programmatic access using access key ID and secreat access key combination.
Step 1.3: Choose to attach existing policies directly to this user.
Step 1.4: Attach the pre-defined policy AWSApplicationDiscoveryAgentAccess to this user.
Step 1.5: Download the Access Key ID and the Secret Access Key for this user.
Step 1.6: Open Cloud9 service and then open the IDE.
Step 1.7: Upload the labuser.pem file using the File menu.
Step 1.8: Give read persmissions to root on the pem file using: chmod 400 labuser.pem
Step 1.9: Get the IP address of application server using the aws ec2 describe-instances command and grep "PrivateIPAddress"
Step 2.0: Got IP: 10.16.10.88
Step 2.1: SSH into the instance using\: ssh -i labsuser.pem ubuntu@10.16.10.88
Step 2.2: Run the following commands to download the relevant files:

                cd /home/ubuntu
                curl -o ./aws-discovery-agent.tar.gz https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-        
                2/linux/latest/aws-discovery-agent.tar.gz
                curl -o ./agent.sig https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-2/linux/latest/aws-
                discovery-agent.tar.gz.sig
                curl -o ./discovery.gpg https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-
                2/linux/latest/discovery.gpg
                
Step 2.3: Verify that the agent signature is also fine:

                gpg --no-default-keyring --keyring ./discovery.gpg --verify agent.sig aws-discovery-agent.tar.gz
                
Step 2.4: Unzip the agent tar file using the following command:

                tar -xzf aws-discovery-agent.tar.gz
                
Step 2.5: Install the agent using the following command:

                sudo bash install -r us-west-2 -k <Seccret Key Id> -s <Secret Access Key> -p true -c true -b true

Note the Configuration Settings in the Ghost Application Instance
-----------------------------------------------------------------
Run the following command to see all processes currenty running for all users then pipe and grep for ghost word:

                ps aux | grep ghost
                
You will see ghost being run by ghost user. Go to the root directory and explore.
You can browse to 
                
                root -> ghost-app -> ghost -> config.production.json
                
These are what I found the contents of the config.production.json file:

                {
                    "url": "http://54.244.207.145:2368",
                    "server": {
                    "port": 2368,
                    "host": "0.0.0.0"
                },
                "database": {
                    "connection": {
                    "host": "10.16.11.80",
                    "user": "ghost",
                    "password": "oranges",
                    "database": "ghost_prod"
                }
            },
                "mail": {
                    "transport": "Direct"
                },
                "logging": {
                       "transports": [
                       "file",
                       "stdout"
                    ]
                },
                "process": "systemd",
                "paths": {
                    "contentPath": "/ghost-app/ghost/content"
                 }
                }
                
Step 3.1: Get the public IP address of your web server: got it: 54.244.207.145
Step 3.2: Open a browser and visit the webpage: http://54.244.207.145:2368
Step 3.3: You can play around the environment by going to the /admin page.

Connect to the Database Instance and Install AWS Application Discovery Agent
----------------------------------------------------------------------------
Remember, we got the private IP address of the EC2 instance hosting the database server from the json cconfig file. So now we SSH into this instance:

                ssh -i labsuser.pem ubuntu@10.16.11.80
                
Step 2.2: Run the following commands to download the relevant files:

                cd /home/ubuntu
                curl -o ./aws-discovery-agent.tar.gz https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-        
                2/linux/latest/aws-discovery-agent.tar.gz
                curl -o ./agent.sig https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-2/linux/latest/aws-
                discovery-agent.tar.gz.sig
                curl -o ./discovery.gpg https://s3-us-west-2.amazonaws.com/aws-discovery-agent.us-west-
                2/linux/latest/discovery.gpg

Check that we have a good signature for the discovery agent:

                gpg --no-default-keyring --keyring ./discovery.gpg --verify agent.sig aws-discovery-agent.tar.gz
                
Untar the downloaded file as follows:

                tar -xzf aws-discovery-agent.tar.gz

Install using the following command:

                sudo bash install -r us-west-2 -k <Access Key ID> -s <Secret Access Key> -p true -c true -b true
                
Now explore the file listings to see if MySql is there:
                
                ps aux | grep sql
                
We see that MySql is indeed running on this box. Check the contents of the MySQL configuration file:

                cat /etc/mysql/mysql.conf.d/mysqld.cnf
                
Check to see if you can login using the credentials that you discovered in the json config file on the server:

                mysql -u ghost -p
                
Then enter the password oranges and then use the following command to show the databases:
       
                show databases;
                
Start the Agent Data Collection Process
---------------------------------------

Step 4.1: Go to the AWS Migration Hub service.
Step 4.2: Discover -> Data Collectors -> Agents

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/AWSMigrationHub.png)

Step 4.3: Select agents and start data collection, then disable data collection for Athena.

Step 4.4: Under Discover -> Servers, you should see the two servers that we have installed the agents on. 

You should select each server separately and see the detailed statistics, in particular, the "performance information" which is going to give you a good idea about the current load on each machine which should help you in deciding the type of EC2 instance you want to use to migrate these machines to AWS.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/Servers.png)

Examples of Decisions Based on Info Collected from Agents
---------------------------------------------------------

Some decisions could be e.g.

1. Too much CPU usage on the database server -> use C2 instance
2. Too much RAM usage on application server  -> use R2 instance
3. Too many disk reads, writes               -> IO provisioned EBS volume.

Multi-Environment Security and Communications
---------------------------------------------
1. Always use authenticated communicaiton protocols e.g. IPSec and TLS to ensure data is encrypted in transit.
2. Setting up automated tools to identify attempts to move data outside defined boundaries is very helpful.
3. For data at rest, encryption is the best strategy to ensure appropriate methods are in place. In addition, Key management solutions are also important. For AWS, usually you can just turn on the encryption option.
4. Ensure least privilege, users and apps should have access only to the necessary data required and only for a time period.

Data Considerations when Migrating
----------------------------------
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/dataconsiderations.png)

There are a ton of options depending on whether you have time and or the bandwidth to move your data to AWS.

# Lab 2 

Steps for Migration
-------------------
1. Create a mysqldump.
2. Use AWS CloudFormation to launch a MySQL database and a Ghost application in the target Region.
3. Restore the target database from the mysqldump.
4. Create a snapshot of the Ghost application's Amazon Elatic Block Store (EBS) drive.
5. Mount a new EBS drive that you create from the snapshot.

MySqlDump - is a backup program which is used to dump a database or a colleciton of databases in order for it to be transferred or copied to another location. It usually uses SQL statements to create tables etc. but can also tansfer the database into CSV or XML formats. Remember that stored procedures and events and views are not backed up using it. The three ways to invoke mysqldump for various settings are as follows:

                    shell> mysqldump [options] db_name [tbl_name ...]
                    shell> mysqldump [options] --databases db_name ...
                    shell> mysqldump [options] --all-databases
                    
Step 5.1 - Create an S3 bucket.
Step 5.2 - We need to modify the "/etc/mysql/mysql.conf.d/mysqld.cnf" mysql configuration file to enable bin_logging by enabling the 
Step 5.2 - We need to modify the "/etc/mysql/mysql.conf.d/mysqld.cnf" mysql configuration file to enable bin_logging by enabling the "#server-id = 1" and "#log_bin = /var/log/mysql/mysql-bin.log" lines as otherwise mysql dump will complain. Unfortunately, we cannot directly edit this file, so we will issue the following commands:
        
                    sudo sed -i '/server-id/s/^#//g' /etc/mysql/mysql.conf.d/mysqld.cnf
                    sudo sed -i '/log_bin/s/^#//g' /etc/mysql/mysql.conf.d/mysqld.cnf
                    
The "sed" command in Linux is typically used to replace strings from within a file.                
Restart mysql service using the following:
    
                    sudo service mysql restart
                    
Step 5.3 - Go to the root directory and issue the following command:

                    sudo mysqldump --databases ghost_prod --master-data=2 --single-transaction --order-by-primary -r 
                    backup.sql -u ghost -p

So now backup.sql contains the backup database.

Step 5.4 - Copy this backup sql file to the S3 buket using the following command:

                    aws s3 cp backup.sql s3://myBucketName
                    
Noticed an interesting property of S3 that if you upload the same name file from CLI, it directly overwrites the old file.
                    
Preparing a Target Environment for the SQL Server and Ghost Server Using CloudFormation Templates
-------------------------------------------------------------------------------------------------
Step 6.1 - Go to the dashboard and change your region to US-East (North Virginia)
Step 6.2 - Go to EC2 -> Network & Security -> Key Pairs -> Generate Key pairs
Step 6.3 - Go to CloudFormation then create a new stack and select the following given URL as the source template:

                    https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-Migration/lab-2-conventional/us-east-
                    1.template

Step - 6.4 Name the stack as clone-stack and proceed with defaults.

Restore your Database
--------------------
Step - 6.5 Login to the east machine.
Step - 6.6 Download the backup.sql file (which is the database dump) to this machine using the following:

                    aws s3 cp s3://2010-usman-sqlbackup/backup.sql /home/ubuntu/backup.sql
                    
Step - 6.7 Start mysql using the following command:

                    mysql -u ghost -p
                    
Step - 6.8 Show the current databases and the tables within ghost_prod db and it will be a blank database with no tables:

                    show databases;
                    use ghost_prod;
                    show tables;
                    
Step - 6.9 Now give backup.sql as the source of the database i.e. "Seed the database".

                    source backup.sql;
                    
Now issuing a show tables command will show all the tables within the database. The important point to note is that you must first go inside the relevant database BEFORE seeding it using "use db_name;" command.

Backup and Restore the Ghost Application
----------------------------------------
Basically, all that you have to do is to create an EBS snapshot, then move it to the correct region, and then populate the ghost application instance with the data that it contains.

# Week 3

Amazon EBS is a block-level volume system which you can attach to EC2 instances. You can create a file system on top of these block level volumes. 

Elastic Block Sotrage (EBS)
---------------------------
EBS offers two types of volumes:

                    EBS Volume Types:
                    
                    1. SSD:
                        1.1 General Purpose
                        1.2 Provisioned IOPS (Highest performance levels, real-time performance)
                        
                    2. HDD (Hard Disk Drive) -> Cost effective
                        2.1 Throughput-Optimized (Lots of accessed) -> when you need low cost but higher throughput
                        2.2 Cold HDD has lower throughput

Elastic File System (EFS)
-------------------------
It is a file sytem that allows you to create file systems which you can for creating files etc and which you can mount onto EC2 instances. 
                     Differences between EBS and EFS  
                     
                      EFS allows you to have shared file system among multiple servers.
                      AWS housed EFS can be mounted to on-premise servers, which can be immensely useful in migrations.
                      
S3
--
Object based storage for storing objects and also contains meta-data. 
S3 allows you to access or works with objects through web console and programmatic access.
S3 can also help store images, backups and other dumps.

AWS Smowball
------------
AWS Snowball allows for moving 80TB of data directly to AWS and also has KMS based encryption capabilities.
There is another option called snowmobile which is huge and provides for moving huge amounts of data.

AWS Storage Gateway
-------------------
AWS Storage Gateway utilizes a VM image in your on-prem environment that SECURELY connects to a gateway endpoint in your AWS allowing for connecting your local storage resources to AWS.

                    Storage Gateway Offers the following options:
                    
                    1. File based.   -> Allows a file based interface into S3 allows to store/retrieve files
                    2. Volume based  -> As iScsi devices 
                          2.1. Cached Volumes: Store your data in S3 and cache frequently accessed data
                          2.2 Stored Volumes: Data stored locally and backup to AWS
                    3. Storage based -> Tape gateway provides a virtual tape interface.
                    
AWS Data Sync
-------------
Simplifies and automates the task of moving data during migrations. It uses an agent based approach and you can use the data sync console to manage the data migration.

AWS DMS (Database Migration Service)
------------------------------------
You can use DMS to migrate to and from most modern databases, allows every combination e.g. Oracle to Oracle, MySQL or MySQL. 

DMS allows for "ongoing replication" with zero downtime.

DMS even allows for converting stored procedures etc. into the appropriate target platform.

DMS has an interesting tool for schema conversion from one database to another. 

AWS Schema Conversion Tool
-------------------------
Allows for converting schemas from one database to schemas for another database.

Amazon Aurora Database
----------------------
Is a MySQL and PostGres compatible database within Amazon Relational Database Service. Provides multitudes of speed in throughput and meets the highest levels of performance. 

                    Aurora Properties:
                    1. Several times higher throughput then traditonal databases
                    2. Fault tolerant and self healing bad disk blocks in the background
                    
Amazon Aurora Serverless
------------------------
Is a configuration of Aurora is an auto-scaling solution in which the db will automatically start, shut and scale out and in according to the situation. This can be very helpful in migrations in which you want to optimize db performance in the cloud.

                    Aurora traditional -> Well known data workloads
                    Aurora Serverless  -> Unknown workloads which scales dynamically
                    
Options to Connect On-premise network to AWS VPC and Services
-------------------------------------------------------------------------------
                    
                    1. VPN over the Internet -> Makes a secure VPN tunnel between on-prem network and AWS
                    2. Direct Connect -> Direct connection to AWS without going to the Internet
                    3. VPN over Direct Connect -> Makes it even more secure as its over private network and also secured
                    
                    
# LAB 3
We are tasked with migrating a database from on-premise to AWS and while doing this, we will replatform the MySQL database to Aurora using the Amazone Database Migration Service (DMS).

Step 7.1 - Create an Aurora database in the target region.
Step 7.2 - Open DMS and create replication instance called ghost-db-replication.
Step 7.3 - Basically in this step, we create an endpoint with mysql on prem server as the source. Then create an end-point then choose source end-point, give source RDS instance IP as source IP and mysql as the source database and set username and passwords as we know already i.e. ghost and oranges.
Step 7.4 - Basically now we create a destination end point which is the RDS instance that we created in the last step. So we select the RDS instance as the destination endpoint.


Step 7.4 - 

Deployment Strategies
---------------------
Red-Black - Means our system is running both on premises and on the cloud as well simultaneously. You have setup DNS entries for both and data is being replicated. Then you finally switch to the AWS setup but keep the on-prem running as well. If everything is well then great, otherwise you change the DNS entry to move back to on-prem.

Blue-Gree: Same as above, but the cut-over is gradual not instantneous i.e. you gradually move. Wih red-black if there is anhy problem you may experience downtime but with blue-green you do it gradually, so you can gradually test it out. One limitation is that your DNS should support little by little shifting of load between on-prem and on AWS. Second, it is more complex.

Automating Migration
--------------------
Most of the tools that we have seen so far need manual work from the console which will make migration difficult, error prone and difficult to predict in terms of time requirements. Luckily, AWS provides API centric approach and the key idea is that if we could somehow automate the task of migration, we can scale the migration exponentially and reap tremendous benefits.

Accessing AWS APIs
------------------

                    1. Use various Software Development Kits (SDKs)
                    2. Use AWS Command Line Interface (CLI)

These APIs allow to use scripts and applications to automate a lot of the migration tasks. It is absolutely necessary to become second nature with these APIs if you want to be able to use AWS for anything serious.

AWS Cloud Formation
-------------------
Infrastructure automation tool that allows code -> infrastructure. It offers templates which you can use to create, modify and update your infrastructure. This capability is very important during migrations as it allows you to quickly environments which are exactly like your production environment. You can quickly spin up complete environments, modify it and tear it down when required.

AWS System Manager
------------------
Manages servers e.g. running a single command across a fleet of servers. Interestingly it allows not only control over servers in AWS but also control over servers running on on-premise servers by installing agents. Thus, you have a single window of view into all your servers running whether on prem or on the cloud.

TSO Logic Solution
-------------------
We have looked at a lot of tools which help us in migration. Now we focus on a single solution called TSO logic which allows us to manage ALL of these services from a single point and provides insights into what you have and what is happening on the destination. Best tool to use. Allows visibility into what you are running in your computer, database, storage and other aspects. Also tells the total cost of ownership to make informed decisions as it also includes predictive analytics.

Cloud Endure
------------
One of the most powerful tool at our disposal to manage migrations at large scale. A fine-tuned machine which makes end-to-end migration into AWS. Godo for large scale migrations, automatically converts a full running application from on prem to AWS. Has machine conversion and orchestration services.


# LAB 4

In this lab, we will use Cloud Endure for migration.

Step 1.1 - Create an account on Cloud Endure, note this is an AWS sibsidiary company,
Step 1.2 - Create a new policy -> JSON, then paste the following:

           https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-Migration/lab-4-cloud-endure/iampolicy.json
           
Step 1.3 - Create an IAM user and attach the policy. Copy the Access Key ID and the Secret Access Key (AKIAWK5BBEXGFVL7TYU6 , a67dtwN8Yo0wBA3E5NeCqSiw4tbbD80dzuOxVITP)

Step 1.4 - Create a new migration project in CloudEndure and provide the Access Key ID and Secret Access key of IAM user.
Step 1.5 - Now you need to specify migration source, so that will be US-west-Oregon.
Step 1.6 - Now you need to spcify the migration target, so select EU (Frankfurt).
Step 1.7 - Install CloudEndure agents on the machines.
Step 1.8 - Open Cloud9 IDE for US-West (Oregon) region.
Step 1.9 - find the private IP address of your application machine using the following command:

            aws ec2 describe-instances --filters "Name=tag:Name,Values=ApplicationInstance" | grep -i -m 1 
            "PrivateIpAddress"

Step 2.0 Shell into the IP address using the key.
Step 2.1 Find out the public IP of your ghost machine:

            curl http://169.254.169.254/latest/meta-data/public-ipv4
            
Step 2.2 - Open the IP:2368 in a browser to see the ghost web application.
Step 2.3 - You want to make some changes in the web application as well as the database so that when you migrate, you are sure that these changes are also migrated. Go to the admin portal:

            44.226.134.99:2368/admin
            
Install CloudEndure agent on the application instance
-----------------------------------------------------

Step 3.1 For some reason, before migration, you need to remove the userdata associated with the instance:

            sudo rm /var/lib/cloud/instances/<yourinstanceID>/user-data.txt*
            


Step 3.2 - Download the CloudEndure agent:

            wget -O ./installer_linux.py https://console.cloudendure.com/installer_linux.py
            
Step 3.3 - Remember to have selected "Show me How" in the cloudendure console when creating project. You will get the actual instructions to launch this installer on that page:

            sudo python ./installer_linux.py -t D694-1190-123B-C401-120A-45BA-FE41-8736-A5B1-1823-9FAB-6E2A-5DB9-76DA-6CF5-
            9A8E --no-prompt
            
Install CloudEndure Agent on the Database instance
--------------------------------------------------
Step 4.1 - SSH into the database instance.
Step 4.2 - Remove userdata from the instance using the instance ID and the following command:
           
           sudo rm /var/lib/cloud/instances/<yourinstanceID>/user-data.txt*

Step 4.3 - Download the cloudendure agent:

           wget -O ./installer_linux.py https://console.cloudendure.com/installer_linux.py
           
Step 4.4 - The data replication process progresses automatically on both the app machine and the database machine as soon as the agents are installed. You can check the process in the CloudEndure console.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Migration/cloudendure.png)

Step 4.5 - Once the replication is done, you can then proceed with the testing phase.
Step 4.6 - Select both the machines, you have two options: "Test" or the "Cut Over" then wait for the rockets to turn purple
Step 4.7 - Head over to the EC2 instance then select application instance (note down VPC and subnet IDs)
            VPC: vpc-0561a1954a75c85cc
            Subnet: subnet-0c2a40a8e493d398b
Step 4.8 - Go back to Cloud9 -> Create Environment , name it Test


# Wrap Up
Keep in mind that don't try to force one single migration solution (6Rs) for all your applications on all servers and instead, study each application individually.
