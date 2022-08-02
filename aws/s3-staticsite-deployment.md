1. Create s3 bucket
2. Do no add encryption
3. Make it public. Keep it owned by owner.
4. Add Policy to GetObjectRead
5. ```
6. {
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
7. ```