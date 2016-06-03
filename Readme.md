# LambdaTranscoderTrigger

A Simple set of AWS Lambda functions for automatically triggering Elastic Transcoder when a video arrives in an S3
 bucket.  Its so simple that I just put both functions in the same file, you have to create two lambda functions
 in the console with different handlers:
 
 1. trigger_elastic_transcoder.start_et_handler : Tied to CREATE events on your S3 bucket
 2. trigger_elastic_transcoder.delete_source_after_et_finished_handler : Tied to your ET's SNS topic
 
## Setup

### The Lambda Policy Document

```json

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::BUCKETNAME/*"
            ]
        },
        {
            "Sid": "2",
            "Effect": "Allow",
            "Action": "ses:SendEmail",
            "Resource": "*"
        },
        {
            "Sid": "3",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Sid": "4",
            "Effect": "Allow",
            "Action": "elastictranscoder:CreateJob",
            "Resource": "*"
        }
    ]
}


```


### The various services

Here's my abbreviated notes of what to set up - they kinda assume you already know how AWS and Elastic Transcoder work,
and are here mainly to refresh my memory when setting up new accounts

1. Create the s3 bucket and write down its name/arn
2. Create the lambda policy (See Above)
3. Create the lambda role that has that policy and write down its arn
4. Create a SNS topic that will be used for notifications, write down its arn
..* Optional: Add/Confirm your email as a subscription for debugging (for now)
5. Create a Elastic Transcoder pipeline for use:
..1. Name: user-video-conversion
..2. Bucket: The bucket you created
..3. IAM Role: Create console default
..*
..4. Config for transcoded files
..5. Bucket: The bucket you created
..6. Storage class: standard
..7. (Same for thumbnails)
..*
..8. Expand notifications
..9. Set warning, completion, error all to the arn we created above (we could create different ones but not going to here)
..10. can set progress too if you like
..*
..11. Encryption
..12. Leave alone
..13. Hit create pipeline
... Write down the pipeline ARN and ID
6. Put the bucket name, prefixes, and pipeline ID (NOT ARN) into the Lambda function header (See below)
7. Create both Lambda functions and paste in the code
8. Profit!

 
### In the Lambda function itself
 At the top of the code, change these variables (Before you paste into lambda):
 bucket_name
 unconverted_prefix
 converted_prefix
 thumbnail_prefix
 pipeline_id
 
 You can also change these, but less likely you need to:
 preset_id='1351620000001-100070' # This is System Preset:Web, basically MP4 max size 1280x720
 region_name='us-east-1'
 delete_upon_completion_enabled=True



 