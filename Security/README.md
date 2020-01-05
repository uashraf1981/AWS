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
                  

2. Incident Response Plan
-------------------------


3. Automated alerting and remediation
-------------------------------------
