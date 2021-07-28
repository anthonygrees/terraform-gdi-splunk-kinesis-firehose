# Terraform "Get Data In" to Configure CloudWatch Logs to Splunk via Kinesis Firehose
  
### About
This terraform configures a Kinesis Firehose, sets up a subscription for a desired CloudWatch Log Group to the Firehose, and sends the log data to Splunk via a HEC over SSL.  A Lambda function is required to transform the CloudWatch Log data from "CloudWatch compressed format" to a format compatible with Splunk.  This terraform also takes care of configuring this Lambda function.
  
### What You Need
You will need the following:
- Splunk with SSL enabled (No self signed certs as Kinesis won't accept it)
- HEC Token from your Splunk administrator
- Splunk AWS Addon app installed
- HEC configured with `Enable indexer acknowledgement` ticked
Note: This code does not use KMS yet.  I plan to add it for security.
  
### Example `tfvars` File
```bash
region = "eu-north-1"
aws_region = "eu-north-1"
arn_cloudwatch_logs_to_ship = "arn:aws:logs:eu-north-1:<your_account>:log-group:/aws/kinesisfirehose/kinesis-firehose-to-splunk:*"  
name_cloudwatch_logs_to_ship = "/aws/kinesisfirehose/kinesis-firehose-to-splunk"
hec_token = "999999a-XXXX-9999-XXXXX-9999XXXX"
hec_url = "https://your_url:8088"
s3_bucket_name = "your_bucket_name"
```
  
### How to Run it
```bash
terraform init


terraform apply


Apply complete! Resources: 15 added, 0 changed, 0 destroyed.

Outputs:

cloudwatch_to_firehose_trust_arn = arn:aws:iam::<your_account>:role/CloudWatchToSplunkFirehoseTrust
destination_firehose_arn = arn:aws:firehose:eu-north-1:<your_account>:deliverystream/kinesis-firehose-to-splunk
```
  
### Test Kinesis Firehose
Go into the Data Firehose and click on the `Test with demo data` tab.  If you do not see your results in Splunk then go check the S3 bucket for errors.  (Remember to download the error in the bucket to read the text or you will get an `Access Denied` error in your browser.)
  
![Kinesis](/images/kinesis.png)
  
### KinesisFirehose-Policy
Should you get an issue or warnings with the `KinesisFirehose Policy` then you can open up the permissions a bit more by replacing the policy with this one.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:ListBucketMultipartUploads",
                "s3:ListBucket",
                "s3:GetObject",
                "s3:GetBucketLocation",
                "s3:AbortMultipartUpload"
            ],
            "Resource": [
                "arn:aws:s3:::<your_bucket_name>/*",
                "arn:aws:s3:::<your_bucket_name>"
            ]
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": [
                "lambda:*",
                "kinesis:*",
                "firehose:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": "logs:PutLogEvents",
            "Resource": "*"
        }
    ]
}
```