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
8. You need to harden the AMI snapshots that are used as base configuration images by deleting the .pem files which contain the private key. More dangerously, you also need to delete history files: AWS config history and the bash history. If you have run AWS config command, you also need to list the hidden .aws folder and the file named credentials since it lists the Access Key ID and the Secret Access Key for any user created. The Bash history (command used is history) also needs to be deleted. So all in all, we need to manage 3 things: 1) Pem Files 2) AWS credentials file within hidden AWS folder 3) Delete bash history and 4) remove authorized_host file
9. You need to keep the servers, machines and applications updated and patched, but make sure that excessive update on one component doesn't break the security in another i.e. you must take care of dependencies. SOLUTION: Test in a test environment.
10. Best practice when one of your system is compromised by malware is to take backup of data and reinstall VM from a fresh baseline image.
