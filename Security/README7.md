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
