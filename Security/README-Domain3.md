# Domain 3: Infrastructure Security

CloudFront
----------
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudfront.png)

Origin: can be 
1. S3 bucket
2. AWS resource
3. Custom source such as on-premise

anything that presents the content using HTTP or HTTPS can be used as a source by CloudFront.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudfrontcustom.png)

* IF you select a custom origin for the CloudFront distribution, e.g. if the origin is your custom domain then you need to configure a lot of other options such as the protocols used (TLS etc.) BETWEEN cloudfront and the origin as shown in the figure above. An interesting option apart from forcing HTTP or HTTPS is the "Match Viewer" protocol in which case this distribution uses whatever protocol the user is using such as HTTP or HTTPS.


User. <--> Edge Location (CloudFront) <--> Origin

if you say Match viewer, then CloudFront will use the same protocol between Origin and itself.

        Two different set of protocols:
        
        1. Viewer protocol policy
        2. Origin protocol policy
        
        You can have the same protocols or different protocols.

lamdda for CloudFront:

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudfrontcustom.png)

* An interesting feature is the Lambda for CloudFront which allows you to execute lambda functions at any time:
  1. When a user request is received at CloudFront called viewer request
  2. Before CloudFront sends a request to origin called origin request
  3. After a response is received from the origin at CloudFront called origin response
  4. Before CloudFront sends a response to user called user response
  
This is very useful cpability to intercept requests in CloudFront and perform compute compute operations e.g. deliver customzied content to user e.g. if a user has macbook pro or other higher resolution system then deliver high resolution pics or low resolution pics for slow system.

* Exam tip: The domain name must match the certificate name if you are using SSL certificates.
* Exam trip: You definitely need certifictes trusted by 3rd parties, you cannot use self-signed certificates on your origin.
