# Key Management Service (KMS)

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/kms.png)

KMS helps with management of keys.

Keys are used together with an encryption algorihtm to encrypt or decrypt data.

CMK = Customer Master Key: It is a logical entity, it is not actually a piece of cryptographic material. It is a pointer to an
actual key. A CMK has a unique ID, so it can be referred to in a unique way.

      * Exam: Once KMS has generated the CMKs, they never leave the KMS and you can only interact with them via KMS.
      * Exam: CMK can only encrypt or decrypt data up to 4 KB in size.
      * Eaxm: KMS meets FIPS level 2. 
      * Exam: By default, no one has access to CMK according to the KMS policy, so root user can only access it.
      
      aws kms create-key --description "My CMK" --region us-east-1

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/keycreation.png)

AWS will go ahead and create a key and return the JSON meta-data, importantly, it contains the Key-ID and Key-Manager field which specifies that it is a customer managed or AWS managed key.

      Two types of Customer Master Keys (CMKs):
      
      1. AWS Managed CMK or
      2. Customer Managed CMK
      
You can check these two types from the console as well since AWS managed keys have the AWS icon and path is aws/key-name.

Alias: Points to a key in a specific region, in the command to creater the alias you need to give the key ID of the actual key

      * Customer Master Key --> generate Data Encryption Key --> Used to encrypt actual data
      
      So whenever encryption is required e.g. by S3, then it generates data encryption keys in two formats:
      
      i)  plaintext data encryption key --> used to encrypt data then discarded
      ii) encrypted data encryption key (encrypted with CMK) which is stored in KMS
      
      Every time that S3 e.g. needs go decrypt data, it sends the data encryption key to KMS which decrypts it using CMK
      and returns the decrypted data encryption key
      
      * Exam: Interetingly, the encrypted data key is stored with data itself.
      * Exam: Interestingly, the id of the CMK used to encrypt this data key is also stored within the encrypted key so we
      don't need to tell KMS which CMK is to be used to decrypt this encrypted data encryption key.
      * Remeber, these keys are base64 encoded.
      
      * CMK = One or more actual Backing keys which are the real keys. CMK is a container since backing keys can be created
      new ones, but the old ones stay stored in the CMK since someone may need the old backing keyh to decrypt.
      
AWS Managed Customer Master Keys: AWS manages everything, you don't have access to key rotiation, expiration etc.
Customer Manager Customer Master Keys: Customer can specify its own policies in terms of key rotation, expiration etc and is usually required only in the case of compliance requirements.

      * Key rotation = Create new backing keys while keeping the old ones within the CMK container.
      
      * Key rotation is automatic in AWS, but for customer managed CMK, you generte a new key, then point the alias to the 
      new key.
      
      * Exam: Re-encrypt operation is when we ask KMS to decrypt some data, encrypt with a new key and send us that key
      this allows for re-encrypting data without actually seeing the decrypted data.
