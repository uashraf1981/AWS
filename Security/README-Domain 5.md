# Domain 5


# Kay Management System (KMS)
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmssecurity.png)

KMS handles all aspects related to managing the keys including generation, rotation, deletion and safe storage of them.

The main job of KMS is to generate Customer Master Keys (CMKs) which are a logical representation. CMKs never leave KMS and never leave a region. The ONLY way that you can interact with them is via the KMS.

KMS can only encrypt or decrypt data of up to 4KB size.

KMS is a hardware based service and is compliant with FIPS 140-2. It is a standard to assess and improve cryptographic modules.

CMK is controlled by resource policies or identity policies.

No one has access to the key policy. When you create a new CMK, there is a default policy that applies and which does one thing only i.e. it trusts the account that the key is created in i.e by default the root is a trusted user.

CMK basically generates dataencryption keys. When it calls the GenerateDataKey API, it generates an encrypted data key and a plaintext key. The plaintext version is used to encrypt dqata and then discarded. The encrypted version is stored with the data and decrypted using KMS when required and then this decryupted DEK is used to decrypt the data.

Remember, the DEK is converted into base 64 encoding so you will need to convert it back when decrypting i.e. from based 64 to decimnal, then decrypt.

We don't need to specify which key to use to decrypt this data key as the key has this information embedded within the DKE.

* Exam Tip: CMK is a logical construct, the actual cryptographic thing behind is the backing key. 
** THe CMK is a wrapper for the Backing Key.
* You can rotate the backing keys by generating news ones. The important point to understand is that the old backing keys are not deleted, but stay linked to the CMK wrapper and if any decryption request comes later for something that was encrypted using the old backing key, then that backing key is used to decrypt the data.


CMKs come in two flavors:

1. AWS backed CMK - Customer cannot do much with the key, rotation is enabled once every 1035 days, we cannot disable it
2. Customer Manaed CMK - Update policies, enable key rotation - we can do it once per year at best, not more, old keys are 
                         backed up and stay linked with the CMK.
                         
Noramlly we use AWS backed CMK but in case of compliance or have more control over key expiration timers, then for governance related stuff, in that case we can generate our own keys.

Envelop Encryption:
This concept of using CMK to generate/encrypt data keys. Advantage is that you can store the data keys with the objects as object meta data and that decreases the management overhead. As long as the CMK are secure (and they are since they are within AWS). 

One option is that due to governance requirements, you may have to re-encrypt your data, but in this case, you can simply re-encrypt the data keys without the time consuming process of re-encrypting the huge data.

* Admins can send data to KMS to encrypt without knoeing the actual data.

** Encryption Context: Is an additional layer of security. Basically, AWS saves that where is the encrypted object currentluy saved, along with it's names etc. and if anything in that changes, then it denies the decryption request for security purposes. For example, if the encrypted object is moved or renamed etc. then decryption denied. Implemented through key, value pairs passed along with encryption/decryption requests.

Grant: Is like pre-signed URL, if you want to give a service or application short-term permission to use some particular CMK etc then you can use grant. Otherwise, for long-term permissions, it is better to use resource or permission policies. 

** Grants can be retired as well.

** Deleting Keys: You cannot delete key instantly. You have to schedule key deletion, default is 30 days, but you can specify 7 - 30 days. This is a safety measure since once CMK is gone, it is gone and there is no way of recovering your data.

# KMS in a Multi-Account Setting

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmsinmultiaccount.png)

Whenever a key is created, it has a default key policy attached to it which gives the root user of the account the ability to run any KMW operation.

To allow other accounts access to this key, we add the policy here. Generally, we prefer not to give open access for all KMS operations, but can specify specific actions such as encrypt, decrypt, generate data key etc.

* Remember: Key usage and key access are separate areas in the key policy document.

** Remember, the external accounts will not see the key in any of their drop down menus. You need to give them your account id and the key ID or the key alias so that they can interact with our CMKs. 

Why we need multi-accounts for CMK? perhaps we want to isolate the security/key account which supports the cryptographic operations.


# Cloud HSM

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudhsm.png)

Cloud HSM is a dedicated HSM which reuns in your VPC. It is still a dedicated HSM, but then you don't have to worry about buying the device, managing the device or handling failures.

1. CloudHSM is a decicated hardware in your VPC ---- in KMS, you are sharing the infrastructure with other customers.

2. You create CloudHSM clusters with multiple.  ---- KMS has high availability and high resilience 
instances for HA e.g. put one instance in  
every availability zone inside that VPC. The
cluster runs outside your VPC, but you have
ENI interfaces attached to these clusters and 
these ENIs are inside your VPC.

3. Unlike KMS, AWS do not have access to        ---- AWS has access to KMS, so you can ask them for help.
cloudHSMs so you cannot ask them to help
 
4. Supports FIPS 140-3                          ---- Only supports FIPS 140-2

5. Does not natively integrates with AWS.       ---- Natively integrates with AWS. You can access KMS APIs, endpoints etc.

* When setting up Cloud HSM, you have to set up certificates. Basically, you generate private key and then use it to self-signed certificates.

# Troubleshooting KMS Permissions

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmspermissions.png)

* CMKs are logical entities which represent backing keys.
* Every CMK when you create it, had a default key policy attached to it.
* The only trust that exists between the CMK key and the account is the default key policy, if removed then we cannot access.

* Exam Tip: IAM can delegate the rights of the key to other roles or accounts for management purposes.

So the CMK trusts account due to key policy ---> Account through its IAM gives permissions to other roles or accounts 
*** But you CAN also add these users or roles directly in the key policy instead of the IAM doing it.

** Exam: It is possible to lock yourself out of your CMK if you remove the default key policy.

** Remember there are two sets of permissions when deadling with customer master keys:
a. Admin: create, delete, schedule key deletion, rotate, enable, disable.
b. Users: Usage operations including encrypt, decrypt, re-encrypt, generatedatakeys, creategrant, listgrant, revokegrant.

# KMS Limits

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmslimits.png)

* 1000 CMKs per region
* 1100 aliases per account
* 2500 grants per CMK

Rate Limits:
* 5500 API operations for KMS (encrypt, decrypt) except for us-east, us-west, eu-west which have 10,000 limit.

* In case you exceed these limits, you will get a *ThrottlingException* error.

* Exam Tip: Basically this throttlingerror occurs due to KMS even if you using other services since those services must be interacting with kms.

**** Exam Tip: In cross-account architecture, throttling error occurs in the account that hosts the APPLICATIOAN NOT the CMK i.e. the account where the APIs are being used, but not where the CMK is located so an easy solution is to have one account store the keys and other separate accounts access them, in that way, every accessing account will have its own quota of limits.

# Data at Rest: KMS

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmsdataatrest.png)

KMS provides encryption services to so many other services including DynamoDB, RDS, EBS, S3.

** Exam Tip: The responsibility of KMS ends at giving data key to the service. So for example, KMS keeps the CMK and gives data key to the service and forgets about that data key. When hypervisor wants to decrypt that EBS volume, it retrieves the encrypted data key stored along with the volume and sends to KMS which decrypts it and sends back the decrypted key. THe hypervisor holds that plaintext in memory and kept there as long as the instance is running.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/dynamodb.png)

If you have setup DynamoDb to be encrypted so the CMK is used to generate data key which is used to encrypt a table every time that a table is created in this region. This key is called the Tablekey. Every itemn inside the table is encrypted with a data key and encrypted with the table key end stored with the item.

So   CMK -> Table Key -> Data Key

Table keys are kept for up to 12 hours in memory so even if KMS is offline, the dynamodb will keep on working.
dybamodb pings KMS every 5 minutes to check if permissions for keys have changed.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmsrds.png)

RDS:
Uses EC2 instances and EBS volumes for storage so exactly works as the EBS with KMS i.e. encryption keys for volume encryption are generated using CMK. The hypervisor maintains a copy of the encryption key in the memory.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kmss3.png)

S3 is same i.e. CMK -> datakeys which are used to encrypt the object and then stored with the object as metas-data.

# Data at Rest: Server Side Encryption with SSE-C

SSE-C = S3 feature in which S3 manages encryption but does NOT use KMS nor does it manage the keys, just uses customer provided keys. So basically everytime that a customer puts an object in the bucket, he needs to provide keys.
This refers to Server Side Encryption with Customer Provided Keys.

* The customer key used to encrypt the object is immediately destroyed after encryption but a HMAC value of the encryption key is still stored in the bucket for record purposes and when a key is supplied, S3 will compares the HMAC of that key with the one that it has stored to verify that the key being supplied to decrypt object is the same as was used originally,
* Customer can select the option in which S3 compares that they customer supplied key was not damaged in transit.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/SSE-C.png)


* Exam Tip: Using customer provided keys prevents cross-region replication.

* For end users, you can generate a pre-signed URL which contains the key so that the customer can easily access objects.

The customer supplied keys approach is used only for governance or security policy related needs otherwise typically not used.

** If object versioning is enabled, then you need to manage individual keys for the different object versions.

** Another instance when we may need to use customer provided keys is if we use our on-premise HSM or we want to use cloud HSM.

# AWS Certificate Manager (ACM)

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/acm.png)

* Provides X.509 vs 3 TLS/SSL certificates. Uses pulick key cryptography where one half is secret and other is public.
* No cost of using ACM, just the cost of usage of resources, if any

* ACM easily integrates with lots of different services.

* Certificates renew automatically.

* Certificates can only be allowed to service within the same region.

* Certificates are never stored ununecrypted, they are encrypted using KMS.


# AWS Encryption SDKs

* AWS encryption SDK is an encryption library that makes it easy to use KMS.
* It takes away a lot of heavy lifting and provides code to operations like encrypt, decrypt etc
* One interesting trick is that it has "DATA KEY CACHING" i.e. you can cache keys and then re-use them instead of requesting again from AWS which has a limit on these requests.

# Compliance Examples

AWS Artifact: AWS make available an ACL of documents relevant to compliancce. Any control documents that AWS has for these compliance standards, which you can download, read and click acceptance.
* 

