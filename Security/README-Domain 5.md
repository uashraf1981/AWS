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


