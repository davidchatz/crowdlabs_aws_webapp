# EC2 Web Application - Lab 015 - CloudTrail

**Incomplete - This is still in development and can be ignored as cloudtrail is configured using the wizard in lab-010**

## Goal

As early as possible configure CloudTrail to track all API activity in this (and future) accounts.

This lab goes beyond the default to all generate a audit user and group that has access to the CloudTrail logs using a KMS managed key. The benefit of this approach is in order to access the logs the user must have both CloudTrail permissions *and* access to this KMS key. Any other user with CloudTrail access will not be able to decrypt and read these logs.

The CloudTrail configuration from [lab-010](../lab-010-master-account) can be left in place until this is functioning correctly before being disabled.

## Prereqs
* Master account has been [created](../lab-01-master-account)

## Completed State
* S3 bucket for CloudTrail created
* KMS key for encrypting logs generated
* IAM Audit group defined
* IAM user in audit group created
* Enable CloudTrail
  * All Regions
  * KMS encryption
  * File validation on

# CloudTrail

**Principle**: Secure and minimise use of the root account.

**Principle**: Segregate roles and responsibilities.

One of the great strengths of AWS is the fined grained controls that allows you to define exactly what resources someone (or something) can interact with, and their level of interaction. So when it comes to CloudTrail logs, since they capture every AWS API request, the set of roles (and therefore people) who should be able to see these logs is quite small.

This lab assumes that you will want to establish an audit function, a group/role that has readonly access to CloudTrail and other resources.

The challenge with the fine grained access controls is the complexity they create; it is significantly harder to diagnose why someone can or cannot access a resource as policies can be defined at the organisation, IAM and resource level.

## Create Audit Group

* Navigate to *IAM*
* Create a group called *group-audit*
* Add the AWS ReadOnly policy to the group


## Create Audit User


## Create S3 bucket

By creating the bucket you get to determine which region the budget is in; the CloudTrail console wizard does not give you this option.

* Navigate to **S3**
* Select **Create Bucket**
* Name the bucket using your global prefix, in this example I used *cle2wa-s3-trail-master*
* Select a region
* Enable *Versioning* 



```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSCloudTrailAclCheck20150319",
            "Effect": "Allow",
            "Principal": {"Service": "cloudtrail.amazonaws.com"},
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::TRAILBUCKET"
        },
        {
            "Sid": "AWSCloudTrailWrite20150319",
            "Effect": "Allow",
            "Principal": {"Service": "cloudtrail.amazonaws.com"},
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::TRAILBUCKET/AWSLogs/ACCOUNTID/*",
            "Condition": {"StringEquals": {"s3:x-amz-acl": "bucket-owner-full-control"}}
        }
    ]
}
```

## Create Customer Managed Key (CMK)


## Configure Access to Key



## Enable Cloud Trail

TBD: Replace this using the resources configured above

*Note: This may generate costs to you*

I recommend setting up CloudTrail immediately so that you capture everything you do from this point on. This is especially handy if something does not work and you don't understand why.
* Navigate to **CloudTrail**
* Create a trail called `trail-master`
  * See the [Style Guide](../../STYLEGUIDE.md) about establishing a naming convention.
* **Apply trail to all regions**
  * Although we disable access to other regions below, they could be re-activated later and it does not cost anything to enable for all regions if there are no API calls in those other regions.
* Enable all **Read/Write events**
* For this account we should not require logging for S3 data events, so ignore that section
* Create a new bucket and specify a unique name
  * Prefix the bucket name with the global prefix you selected to make it unique
  * This course will use the same bucket for CloudTrail logs from all accounts, so `cle2wa-trail-bucket`.
* Select **Advanced** and check that
  * **Enable log file validation** is on
* Press **Create**

# References

[S3 Bucket policy for CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html)

[Encrypting CloudTrail with KMS](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html)

[IAM Policies for KMS](http://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html)

# Possible Enhancements
* Enabling versioning and logging on the cloud trail bucket
* Define tags