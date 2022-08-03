## Wire up cloudfront to s3 and set up https
https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-https-requests-s3/

* go through the instruction
* use http redirect
* use a custom ssl certificate (Register for free from 'AWS Certificate Manager') because we're going to use CNAME routing using route53 instead of default cloudfront name and ssl cert.

AWS Certificate manager will be in validate state until cnames are copied over to Route53, which is going to be part of the validation process. You can just click on 'create recors in route53' in ACM console. 

Create CNAME for both root domain, and sub domain (www.domain.com). Route53 will now route traffic to _5f454061bc3faca55448c72XXXXXXX.wsqgXXXXXX.acm-validations.aws.



* for cloudfront standard logs, use prefix equal to folder name in s3 bucket
also.. some rights needs to be set
```
 If the bucket that stores the logs uses the bucket owner enforced setting for S3 Object Ownership to disable ACLs, CloudFront cannot write logs to the bucket. For more information, see Controlling ownership of objects and disabling ACLs for your bucket.
```
* when using cloudfront, you can enable 'block public access' for s3 bucket. Cloud front will generate the policies required when it creates it by selecting OAI for bucket access

(may need to clear existing s3 bucket public policy for above to work)

* make sure to set root object as index.html, and (can now disable static s3 hosting),
unless you specified the s3 static website url in the distribution origin instead of the s3 bucket. 

## Route53 link to cloudfront
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-cloudfront-distribution.html

Follow above link. If route53 was linked to s3 bucket, then edit the exisiting ones to the new cloudfront distributions.

make sure to include all the cnames domain.com
and www.domain.com in the cloudfront distribution setting as well as doing up route53 setup.