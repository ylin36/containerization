## directly off s3
Note that hosting directly off s3 will result in bucket needing to be public and no ssl can be applied.

1. Create s3 bucket
2. Do no add encryption
3. Make it public. Keep it owned by owner.
4. Add Policy to GetObjectRead
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::linytech-angular/*"
        }
    ]
}
```

Below is full instruction for hosting route53 to s3 bucket

one limitation is the bucke name must equal to s3. Use cloudfront to work around this limit

https://docs.aws.amazon.com/AmazonS3/latest/userguide/website-hosting-custom-domain-walkthrough.html

see cloudfront-deployment.md