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
