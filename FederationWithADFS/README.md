# Integrating an on premise Microsoft Active Directory with AWS

The basic is idea is that you re-utilize existing credentials/accounts from an on premise account to gain access to AWS instead
of creating all those users again.

Step-1: Create roles in the AWS IAM - ADFS-Admins, ADFS-Developers, ADFS-QA
Step-2: With each role there is a policy attached e.g. for ADFS-Admins the role allows access to all AWS resources
Step-3: On the on-premise Active Directory create security groups with similar names e.g. AWS-Admins, AWS-Developers. Note
you DON'T need to create separate IDs for different people, just put them in approprite groups e.g. all devs in AWS-Developers
Step-4: 
