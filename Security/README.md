# AWS Security Speciality

Domain 1 - Incident Response
----------------------------
Incident has three sub-domains:

1. AWS Abuse Notice
-------------------
AWS teams have automated monitoring tools with which they are constantly monitoring the AWS infrastructure for things like:

    - large changes in patterns, port usage in pattern with known exploits, penetration testing 
    - inspect things internally i.e. attacks againt AWS hypervisors or other internal virtual network infratructure
    - inspect outgoing traffic from your environment e.g. are you doing something illegal or your resource has been hacked and 
    is now being used by someone else
    - manual abuse notices in which someone informs AWS team that your resources are being used illegally
    
Acceptable Use Policy:

a) Cannot use AWS for illegal, harmful, infringement or offensive material
b) Cannot use AWS for sniffing, falsisying origins
c) No Network Abuse - no monitoring and crawling, denial of service attacks, operting open relays an proxys
d) Cannot use AWS for any mass or unsolicited emails or messages

Penetration Testing:

AWS allows penetration testing, load simulation etc but you need to get authorization first.
                  
                  Three main types of events that require prior permission from AWS:
                  
                  i) Simulated Event Testing - e.g. DR simulation, load testing, security training to team, red/blue teaming
                     you just need to send an email to AWS regarding the event
                     
                  ii) Vulerability and Penetration Testing - A web form that you need to fill in and it is pretty expensive 
                  e.g. who you are, contact, systems or targets that you are attempting te. gateways, DNS style attacks, 
                  details of where the pen testing are coming from, IP address who owns those and contact. Overview of what 
                  to expet e.g. start-end dates, who will stop the testing (e.g. if AWS want you to stop the test). AWS only 
                  allows pentests against certain services e.g. EC2, RDS, AURORA, CloudFront, API Gateway, Lambda, 
                  Lightsail, DNS zone walking. Even within these, there are further restrictions e.g. not allowed to 
                  pentesting against EC2 and RDS not allowed to test for small instance types e.g. t instance as these are 
                  always over subscribed and thus your testing can affect other customers.

Four reasons why you may get abuse notice:

                  1) Compromised instance - EC2 may have been compronised and become part of botnet.
                  2) Secondary Abuse - a malware got installed and is calling back home
                  3) Application Function - a genuine app function may get alerted , it may be your genuine app reaching 
                  out to the Internet, but for the automated monitoring tool this may appear as an attack.
                  4) False complaints - You just get false complaints and other AWS users may report. You may have a web 
                  server and it is talking to another server in someone else's resources in their VPC by accident
                  
AWS Abuse Notice:

1. You must respond to the abuse notice
2. You must remove the immedite exploit that has occurred e.g. account credentials being leaked.

                  Steps to Take to Respond to Abuse Notifications - basically be paranoid and risk averse:
                  
                  1. Change the root password and the password to all IAM users.
                  
                  2. Add MFA to all admin users and others who have acces to the console.
                  
                  3. Generate new key pairs for all the EC2 instances by deleting the old key pairs, relaunching the AMI as 
                  a new EC2 instance and edit the .ssh/authorized_keys file which gets populated with the key at the boot   
                  time. AWS uses 2048-bit RSA based public key cryptrography and stores only the public key part. When 
                  launching an instance, you specify the key pair or generate a new key pair. The public key part of this 
                  key gets stored in the .ssh/authorized_keys file on the instance.
                  
                  Important: AWS DO NOT store the private key pairs so if you lose the private key for an instance-store-
                  backed instance, then you need to terminate the instance and launch a new one, and you will loose 
                  data. Interestingly, if however, you loose the keys to an EBS-backed instance, then you still recover it.
                  
                  Important: If you have multiple users which access an EC2 instance, then you can generate their key pairs, 
                  give them their private part of the key and store the public keys in the .ssh/authorized_keys file.
                  
                  4. Delete or rotate compromised IAM keys.
                  
                  5. Delete the following:
                   - unrecognized instances
                   - spot bids
                   - IAM users
                   
                   6. Contact AWS support.
                  
It is pretty common for instances to get compromised through leaked passwords or keys. Often this is due to committing keys to a public repository like github. There are bots that are always scanning public github repositories.
                  

2. Incident Response Plan
-------------------------


3. Automated alerting and remediation
-------------------------------------
