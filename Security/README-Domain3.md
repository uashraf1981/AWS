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
* Exam tip: creating custom SSL certificate for comms between user and cloudfront (edge location) is allowed. you can use ACM (amazon certificate manager) to generate certs.
* Important and Confusing: In the old days, and if you are using old browsers, then every SSL certificate needed a dedicated IP address and this feature can still be used by ticking a checkbox, but it costs around $600/month in AWS. Newer browsers have a feature called SNI. So the client tells the browser which particular DNS name it is looking for and then the browser tells the CloudFront which DNS name it is looking for and then CloudFront can send that particular certificate and also encrypt the session using that particular certificate.

* Remember the above applies only to custom origins, if you use S3 as the origin then these are configured automatically.

Field Level Encryption: IS a nice feature which allows you to encrypt user supplied sensitive info such as usernames and passwords and credit cards at the edge closest to the user so that the communication remains encrypted end to end to the origin server. You can use public and private encryption so that not even CloudFront has access to the information transmitted.

* After creating the distribution you will notice an interesting thing that you can not only access the contents from the distirbution, but YOU CAN ALSO STILL acces the bucket directly and access the contents directly albeit a little slower. You need to secure that by only accepting requesting requests from the cloudfront on that S3 bucket.

        * CloudFront also serves as a validator in the sense that the requests received at the origin are only valid HTTP or 
        HTTPs requests. It is like an edge-based defence against DoS attacks. It can also be configured to use AWS WAF i.e. 
        you put your firewall at the edge, instead of at the CloudFront distribution.

* CloudFront offers several advanced security features such as signed cookies, geo restrictions. It can also integrate with 3rd party components using signed URls and signed cookies.


# Resticting S3 to CloudFront

* We often don't want S3 to serve content directly to customers and instead want only the CloudFront distribution to be able to access the content. 

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/restrictings3tocloudfront.png)

Origin Access Identity (OAI): We can restrict access to S3 to be limited to only the CloudFront distribution by using the Origin Access Identity. It is a virtual identity used by all of CloudFront edge locations for a given distribution.

users (origin id)--- denied-->    S3    <--- allowed -- CloudFront Edge (origin id)

So it should be:   user --> CloudFront (Origin id)--> S3

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/contrastbucketpolicy.png)

The figure above contrasts our existing bucket policy with that one that we should use for limiting access. Note the difference in the principal '*' vs only CloudFront.

        * Interestingly, you can restrict access to S3 directly from the CloudFont 
        console, don't have to do it from S3. You will also need to create an 
        origin id as shown below. Also select update bucket policy automatically. 
        As a last step, you need to remove the * entry for principals from S3 
        bucket policy. As a finishing touch, you may want to upload error.html 
        to the bucket and define the error page to be error.html and also change
        permissions of error.html to be publicly accessible.
        
![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/creatingoriginid.png)

        * Exam Tip: Restricting access directly to S3 may not seem like a big deal
        now since the CloudFront also allows public access, but as we add 
        additional restrictions and security layers to CloudFront distribution, 
        this access becomes important to be blocked.

# Signed URLs and Cookies

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/signedurlsandcookies.png)

Signed URL = Accessing S3 bucket via authenticating using access keys belonging to an IAM user.

        Motivation for using Pre-signed URLs is that in S3 either you can give access to a pcaticular user or more wide 
        ranging access. Similarly, another cumbersome solution would be to define IAM users and give them keys so that they 
        can have access to S3 buckets. To solve these problems, it is just easier to restrict access to certain 
        buckets/objects via pre-signed urls. This is particularly handy for remote workers since we don't have a good and 
        secure channel to transmit the keys securely to those remote users.
        
        Note: Generally pre-signed URLs are used for applications, but they can also be used for people.
        
        * A neat trick is that even if a remote user has no IAM account or anything, you can still generate a pre-signed URL 
        using your own credentials and share that with the remote user who will be able to access the objects in the bucket.
        The nice thing is that you can STILL define how long will that access be and what type of access they have, but 
        essentially, you are still giving them access.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/newsignedurls.png)

We are going to demonstrate that.

Step - 1 Create an EC2 role with policy for S3FullAccess.
Step - 2 Launch an instance, (Amazon linux 2), will configure it to use the above role
Step - 3 Log into the instance and check connectivity by -> aws s3 ls   (should list all buckets)
Step - 4 We will create a bucket using -> aws s3 mb s3://mytestbucket
Step - 5 We will upload a pic -> aws s3 cp ./pictures/mypic.jpg s3://mytestbucket
Step - 6 Enable static web hosting for this bucket. Get the URL For this bucket and suffix that with the pic name, we won't
         be able to access from the browser.
Step - 7 Imagine you have hundreds of objects in this bucket but you only want to prevent access to this particular file. 
         One option is to create IAM user for all those who want access to this object, but in addition to the added 
         complexity, you will also have the problem of how to share the IAM credentials securely with these remote users.
Step - 8 So we generate pre-signed URLs: aws s3 presign s3://mybucket/mypic.jpg --expires-in 60 (this will be seconds) You 
         will get the pre-signed URL which you can copy and share with the remote users. 
         
                The pre-signed URL contains a temporary access key ID and temporary access key included within it
                
                *Exam tip: The URL is signed, so its integrity is guaranteed i.e. no one can edit it without being noticed.
                *Exam: default expiry time for url if not specified is 3600 seconds i.e. one hour.
                
                *Exam Tip: The signed url generated is dependent on the credentials of the entity generating it e.g. in this
                case, the EC2 instance with a role attached to it did that. If we go ahead and remove the policy of S3 full 
                access attacched to this role, then the presigned url will NOT work !! Basically, the role/policy should 
                be there before generating the signed url since it will still need those permissions to access S3 and even 
                you cannot do -> aws s3 ls as you no longer have the necessary credentials, signed-url will also not work.
                
                ** Very important: You can STILL generate a presigned url for object to which you don't have any permissions
                i.e. you can still generate the pre-signed url, but it will just not work and you may be asked in exam that
                how can a pre-signed url generate an access denied message so answer is that pre-signed url is valid if:
                
                1. It is not expired
                
                2. The entity that generated that pre-signed URL has appropriate role and permissions attached for S3
                
                Note: Re-attaching appropriate policy will enable access again including the pre-signed URL
                
                *Exam tip: Since roles typically themselves have temporary credentials therefore it is not a good idea to 
                use them to generate pre-signed URLs and instead proper IAM user accounts should be used to generate the
                pre-signed urls. Otherwise you may be generating urls with very long expiry times but the role that was 
                used to generate the urls has its own credentials refreshed automatically, rendering the url invalid.
                
  With CloudFront distributions, you can use pre-signed URLs and signed cookies. However, before that, you need to define 
  "TrustedSigners" i.e. a list of IAM accounts that can sign the URLs. As soon you define that, the distribution becomes
  private. 
  
        Exam tip: From the moment that you define trustedsigners, the distribution immediately becomes private and now user
        definitely need a signed URL or cookie ito access any object providede by that distribution.

![stack Overflow](https://github.com/uashraf1981/AWS/blob/master/Security/cloudfrontsignedurls.png)

CloudFront has two distribution types:

1. Web based:  Can use both pre-signed URLs and cookies
2. RTMP based: Can use pre-signed URLs only, not cookies

                Exam: Main difference between pre-signed urls and pres-signed cookies is that with pre-signed URLs, you give
                access to a specific object only, whereas with cookies, you can give access to a "type" e.g. jpg so cookies
                are somewhat more flexible. Their downside is that they cannot be used with RTMP distributions.
