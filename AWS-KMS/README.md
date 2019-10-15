# AWS Cloud Hardware Security Module (HSM) and Key Management Service (KMS)

*If you HAVE compliance requirements and want to make sure that only you manage access to your keys, then user AWS CloudHSM*
*If you DON'T have compliance requirements, and want an easier management solution for your keys, then use AWS KMS*

The basic idea behind the *Master Key* is that you can have a data key, but you need to encrypt it, so you can use the KMS to store your Master Key which you use for encrypting your data day.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/IAM-Cross-Account-Access/Delegation%20Cross%20Account.png)


AWS KMS uses *symmetric* keys encoded with AES-256.

AWS KMS is integrated with CloudTrail for all cryptographic operations so that you can track and audit for compliance.
