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
7. Database access must be controlled through IAM policies as well as encryption (KMS). For some DBs like DynamoDB, we also have the option of using VPC endpoints to secure the access database i.e. no need to use public links. We can also encrypt the data before sending to the database, but then we need to reference the encryption keys in queries
