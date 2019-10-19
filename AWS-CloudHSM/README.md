# AWS Cloud Hardware Security Module (AWS CloudHSM)

For key management, traditionally organizations used an on-premise hardware based HSM which automates a lot of stuff including lifecycle management (key creation, export, rotation, destruction and auditing), centralized management, APIs etc. however, this traditional approach doesn’t work well when organizations want to to move to the cloud because:

- The cloud service may or may not have capability of integrating with on premise hardware based HSM 
- Since data has moved to cloud, there may be significant delays to access keys from the on premise HSM
- Multi cloud tenancy can make things difficult

*AWS CloudHSM is often required to meet compliance requirements, where we don't want a 3rd party to manage our Keys*
*You Access and Manage your Keys, AWS has NO Access to your keys*
*CloudHSM supports asymmetric keys*
AWS CloudHSM is a nice solution in which you have a dedicated hardware based HSM in the AWS cloud. AWS just manages the security of that box while you manage the HSM. A nice feature is AWS HSM Clustering in which you have multiple CloudHSM instances in different AZs and they can offer load balancing and key replication.

Clustering CloudHSM
-------------------
If you have a VPC spanning multiple Availability Zones (AZs), then you can host CloudHSM in different AZs inside your VPC and this offers the followig benefits:

1. Key replication among the cluster:- You add keys to one CloudHSM modules, and it gets replicated so keys always available
2. Load Balancing among the CloudHSM modules
3. Offloads the heavy work of aymmetric handshakes or asymmetric authentication from the front end i.e. website, users are sent to HSM then directed back once the handshakes are done

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/AWS-KMS/HSM-Architecture.png)

*Basically use HSM Clustering to make sure that your keys are ALWAYS available*
