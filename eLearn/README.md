# Architecture of the eLearn system

Start Simple
-------------
When we start a new system, the most logical thing to do is to keep it simple because of the following reasons:

1. We want to focus on the core product, high availability and scalability are not immediate concerns and may take the focus away from the core of the application

2. We are not sure if the product will even fly, so we don't want to overprovision or overspend at this stage. If it flys, then sure, we can gradually build high availability and scalability into the platform

We want to start with a single box hosting the LAMP stack. The main issues with this approach are of course:

 - Scale
 - Availability

These concerns will gradually become more important as our customer base grows.

Split Into Tiers
----------------
The logical next step on this path is to split the system into multiple tiers, e.g. 
  
  - Web Server/App Server
  - Database Server
  
Or it can also be divided into 3 tiers. Basically, what we have done is separated each tier into its separate box.

Introduce Redundancy Into the Web Tier
--------------------------------------
The simplest solution to ensure high availability is by adding redundancy e.g. by adding more "boxes" to each tier. It helps alot initially, however, the availability does not grow linearly with adding more boxes eventually. Eventually we will hit a point of diminishing returns.

Expand Redundancy to Other Tiers
--------------------------------
The next logical step is to introduce redundancy to other tiers e.g. database tier. However, please keep in mind that adding redundancy to stateful components such as the webserver or application server with sessions is not straightforward and needs some workarounds e.g. pushing states to the DB in order to make the session data decoupled.

Expand Redundancy for the Data Centers
--------------------------------------
Our next logical step is to introduce redundancy to the data center i.e. host our servers on multiple Availability Zones but within the same region so that the data centers are not that far away from each other so that latency remains low for replication or load-balancing but are far enough not to be affected by a single natural disaster so multi-AZ availability.

Ssalability
-----------
The redundancy approach that we discussed also helps in scalability. Eventually we want to be able to scale out horizontally as well as scale in when load becomes less. This is supported by the auto-scaling features of AWS.


# Data Persistance on EC2 Instances

The data on the instance local hard drive is lost across stopping, reboots or terminations, whereas the data on EBS remains intact across start/stops or reboots and is only destroyed if we terminate the instance. However, it is far safe to host the data on the EBS instance as at least it is there across reboots and terminations and also, we can take snapshots of the EBS volume, or even detach the EBS volume and reattach it to another EC2 instance.

# Backups

Basically we need three different types of backups:

1. Backup of the server machine -> web server -> AMI image backup
2. Backup of the EBS volume attached -> source code files -> EBS snapshot
3. Backup of the datavase -> Database snapshot

AMI snapshots -> encompass the entire EC2 instance including OS, application server, apps, and EBS volumes
EBS snapshots -> snapshot only the EBS volumes (useful if EC2 has multiple EBS volumes only some change)

Most logical is to take AMI snapshots especially if you want a complete image which you can launch immediately in case your primary server shuts down.
