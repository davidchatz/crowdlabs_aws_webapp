# EC2 Web Application - Lab 015 - CloudTrail

## Goal

As early as possible configure CloudTrail to track all API activity in this (and future) accounts.

Using CloudFormation to create the bucket and trail can be done instead of (or even in parallel to) the manual approach using the console in [lab-010](../lab-010-master-account).

## Prereqs
* Master account has been [created](../lab-01-master-account)
* Assumes you have some CloudFormation experience. If not I suggest you do the [Advanced AWS CloudFormation](https://acloud.guru/course/aws-advanced-cloudformation/dashboard) course on [acloud.guru](https://acloud.guru).

## Completed State
* CloudTrail logging all API activity to an S3 bucket

# CloudTrail

**Principle**: Log all activity to enable addition of metrics, alerts, events and retrospective analysis.

## Create S3 bucket

By creating the bucket you get to determine which region the bucket is in, unlike when using the CloudTrail console wizard which does not give you this option.  Some would choose to run this template in us-east-1 since that is where global services like are likely to be.

This bucket will be used by other accounts so we want a name that makes sense for the organisation, but rather than hard coding the bucketname make it a default for a parameter. That gives the user the opportunity to change it and reuse the template in other accounts.

Creating a bucket in CloudFormation is quite straight forward, remember to:
* Set the deletion policy to *Retain* so that the bucket remains even if the stack is deleted
  * This can be annoying when testing the template, since you will have to manually delete the bucket in order to rerun the stack
* Define the allowed pattern for an s3 bucket name so an illegal name is caught before the stack creation starts

Your script may look something like this:

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Parameters:
  trailBucketName:
    Type: String
    MinLength: 3
    MaxLength: 64
    Default: cle2wa-s3-trail-master
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash

Resources: 
  s3TrailMasterBucket: 
    DeletionPolicy: Retain
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: !Ref trailBucketName
      # TBC: should we enable versioning for this?
      VersioningConfiguration:
        Status: Enabled
```

## Define S3 bucket policy

The bucket policy can be taken from the exmaples provided by Amazon in [S3 Bucket policy for CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html).

Because we intend to use this bucket for cloudtrail logs from other accounts, you can either add new accounts to the *s3::PutObject* resources, or remove ths specific account ID from this resource. I have choosen to do the later. 

```yaml
  s3PolicyTrailMasterBucket: 
    Type: "AWS::S3::BucketPolicy"
    Properties: 
      Bucket: !Ref s3TrailMasterBucket
      PolicyDocument: 
        Version: "2012-10-17"
        Statement: 
          - 
            Sid: "AWSCloudTrailAclCheck"
            Effect: "Allow"
            Principal: 
              Service: "cloudtrail.amazonaws.com"
            Action: "s3:GetBucketAcl"
            Resource:
            - !Join ["", ["arn:aws:s3:::", !Ref s3TrailMasterBucket]]
          - 
            Sid: "AWSCloudTrailWrite"
            Effect: "Allow"
            Principal: 
              Service: "cloudtrail.amazonaws.com"
            Action: "s3:PutObject"
            # For a single account this would normally specify the account in the path
            # but since this will be for multiple accounts
            Resource:
            - !Join ["", ["arn:aws:s3:::", !Ref s3TrailMasterBucket, "/AWSLogs/*"]]
            Condition: 
              StringEquals:
                s3:x-amz-acl: "bucket-owner-full-control"
```

## Create the CloudTrail

See [AWS::CloudTrail::Trail](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudtrail-trail.html).

Your trail resource configuration will need to include:
* Reference to the bucket policy and the actual bucket
* Because this trail is logging all regions you also have to include global services
* Output the bucket name

You should end up adding something like [this](scripts/cf-cloudtrail-1.yaml):

```yaml
  trailMaster: 
    DependsOn: 
      - s3PolicyTrailMasterBucket
    Type: "AWS::CloudTrail::Trail"
    Properties: 
      # TBD: How to specify the name of the trail
      S3BucketName: !Ref s3TrailMasterBucket
      IncludeGlobalServiceEvents: true
      IsLogging: true
      IsMultiRegionTrail: true
      EnableLogFileValidation: true

Outputs:
  trailMasterName:
    Description: CloudTrail Bucket
    Value: !Ref s3TrailMasterBucket
```

You should now be ready to run this template.
* Navigate to **CloudFormation**
* Select **Create Stack**
* Use the **Upload a template to S3** and **Choose File**
* Select your script file
* Provide a
  * *Stack Name* (for example *cle2wa-trail-master*),
  * *Baucket Name* (for example *cle2wa-s3-trail-master*)
* Press **Next** and **Next**
* Press **Create**

Once the stack is created successfully, check the S3 bucket, the bucket policy and **CloudTrail** dashboard.

# References

[AWS::CloudTrail::Trail](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudtrail-trail.html)

[S3 Bucket policy for CloudTrail](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html)

# Possible Enhancements
* Extract the bucket creation into a separate template so the trail stack can be deleted and created again without affecting the bucket.
* Enabling versioning and logging on the cloud trail bucket?
* Define and use a KMS key
  * [Encrypting CloudTrail with KMS](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/encrypting-cloudtrail-log-files-with-aws-kms.html)
  * [IAM Policies for KMS](http://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html)
* Define tags