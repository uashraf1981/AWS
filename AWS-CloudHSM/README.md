# AWS Cloud Hardware Security Module (AWS CloudHSM)

For key management, traditionally organizations used an on-premise hardware based HSM which automates a lot of stuff including lifecycle management (key creation, export, rotation, destruction and auditing), centralized management, APIs etc. however, this traditional approach doesn’t work well when organizations want to to move to the cloud because:

The cloud service may or may not have capability of integrating with on premise hardware based HSM
Since data has moved to cloud, there may be significant delays to access keys from the on premise HSM
Multi cloud tenancy can make things difficult
AWS Cloud HSM is a nice solution in which you have a dedicated hardware based HSM in the AWS cloud. AWS just manages the security of that box while you manage the HSM. A nice feature is AWS HSM Clustering in which you have multiple CloudHSM instances in different AZs and they can offer load balancing and key replication

https://github.com/uashraf1981/AWS/blob/master/AWS-KMS/HSM-Architecture.png

The way this is done is that you

what about the delay caused by the latency in the connection between your data on cloud and the HSM on premises since every request may need to access the keys from the on premise HSM for decryption and so so on, introducing unacceptable delays 3.	What if you have multi-cloud tenancy, then you will have a different set of HSM key management tools for the different vendors

So, the next logical solution was to move to

HSM (Hardware Security Module) If required for compliance

The Hardware Security Module is a physical devices on premises. AWS offers AWS Cloud HSM in which AWS has no control, and you have full control. AWS just manages the box. Also, AWS allows placing the HSM inside VPC in multiple AZs and clustered and load balanced and key replication in place i.e. add keys to just one of the Cloud HSM and it gets replicated. This is the perfect solution if your organization requests that keys be placed in dedicated hardware and the clustering approach ensures that keys are always available. So HSM clustering is a nice way of ensuring that your keys are always available.

Another important application of the cloud HSM is if your application is getting bogged down due to asymmetric handshakes then you can offload it to the HSM to handle it before talking to the application.
