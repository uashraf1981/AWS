# AWS Key Management Service (KMS)

*If you HAVE compliance requirements and want to make sure that only you manage access to your keys, then use AWS CloudHSM*

*If you DON'T have compliance requirements, and want an easier management solution for your keys, then use AWS KMS*

The basic idea is that you encrypt your data with the *Data Key*, but then you need to encrypt the data key as well. So you use a *Master Key* to encrypt your data key and AWS KMS stores the Master Key.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/AWS-KMS/Master%20Key.png)

** Remember the scope of keys is always a region **

KMS Master Keys
-------------------
AWS KMS Customer Master Keys (CMK) are *AES-256* *symmetric encoded* keys which never leave KMS and while a client can manipulate the Master Keys, it can never see the actual contents of the key itself. The client can create, delete and apply policies to who has access to Customer Master Keys (CMK). Typically used to encrypt/decrypt data encryption keys. It can encrypt data chunks as well but maximum of 4KB chunks.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/AWS-KMS/Keys.png)

Master keys are of two types:

1. Customer Managed Customer Master Key (CMK): 

2. AWS Managed Customer Master Key (CMK): 

Data Encryption Keys
--------------------
The Data Encryption Keys are use to encrypt the actual data on EBS or EFS e.g. and then the data keys are also encrypted using the Master key and also stored with the data itself.

Monitoring and Compliance
-------------------------
AWS KMS is integrated with CloudTrail for all cryptographic operations so you can deliver files to an Amazon S3 bucket and  which you can later track and audit for compliance.

Programming the AWS KMS API
---------------------------
You can use the AWS KMS API to generate, delete, rotate keys, to encrypt decrypt data (although data keys recommended for data encryption), to encrypt and decrypt data keys and many more.

