# AWS
This repository contains different tasks performed on the AWS platform.

AWS Security Best Practices and Miscellaneous Points
----------------------------------------------------
1. Never use root user login for daily work create administrator users (lacking certain IAM permissions)
2. Enable MFA on root uers account
3. Root user should not have keys and should be deleted if any
4. Never store credentials on EC2 instances or other services, use roles
5. Root user should define password policy e.g. password length, special characters, ageing 
6. If you need S3 without public access, good idea to use VPC end point with direct access to S3 bucket
