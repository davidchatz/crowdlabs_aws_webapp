# EC2 Web App - Lab 035 - Lockdown Subaccount

## Goal
* Lock down the subaccount as was done for the master account
  * Use CloudFormation to get CloudTrail activated and writing to the same buchet as the master account
  * Use CloudFomration to create a console and CLI admin user in the subaccount
  * Use the CLI admin user to lock down the subaccount

## Prereqs
* You can access the subaccount by switch roles from a user in the master account

## Completed State
* Subaccount is locked down
  * Enable CloudTrail
    * Monitoring all regions
    * File validation on
    * Logging to shared bucket with master account
  * 2FA on console admin account
  * Access key created for cli admin account
  * Password policy defined
  * Disable all other regions
  * Set security questions

# Lab 035 - Lockdown Subacconut

## Base stack for master account

Before we do anything else in the subaccount, write a CloudFomration template to enable CloudTrail using the bucket in the master account.
Rather than hard code the bucket name, make it the default of a parameter.
The template should look something like [this](scripts/cf-subaccount-base-1.yaml):

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
  trailMaster: 
    Type: "AWS::CloudTrail::Trail"
    Properties: 
      # TBD: How to specify the name of the trail?
      S3BucketName: !Ref trailBucketName
      IncludeGlobalServiceEvents: true
      IsLogging: true
      IsMultiRegionTrail: true
      EnableLogFileValidation: true
```

Create a stack with this template and verify CloudTrail is enabled and logging.

## Create users to administor account

The rest of the lockdown procedure cannot be implemented through CloudFormation as it does not involve creating a resources. The cli can be used for some of it, but the rest requires the use of the console.

Before we can use the command line interface we will need a user in the account with an access key.

So the first step is to create an admin group, consolve and cli user accounts. This we can do with CloudFormation. 
So extend your template to create an admin group with the AdministratorAccess policy, a console user with a password and a cli user.

The template should look something like [this](scripts/cf-subaccount-base-2.yaml):

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      -
        Label:
          default: CrowdLabs EC2 Web Application Subaccount Base
        Parameters:
          - trailBucketName
          - groupName
          - userName
          - userPassword
          - userCliName
    ParameterLabels:
      trailBucketName:
        default: Name of shared S3 bucket for CloudTrail logs
      groupName:
        default: Admin Group Name
      userName:
        default: Admin Console User Name
      userPassword:
        default: Admin Console User Password
      userCliName:
        default: Admin CLI User Name

Parameters:
  trailBucketName:
    Type: String
    MinLength: 3
    MaxLength: 64
    Default: cle2wa-s3-trail-master
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash
  groupName:
    Type: String
    MinLength: 3
    MaxLength: 64
    Default: group-admin
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash
  userName:
    Type: String
    MinLength: 3
    MaxLength: 64
    Default: admin-console
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash
  userPassword:
    Description: Must conform to password policy
    Type: String
    MinLength: 8
    MaxLength: 32
    NoEcho: true
    ConstraintDescription: Password must be at least 8 characters
  userCliName:
    Type: String
    MinLength: 3
    MaxLength: 64
    Default: admin-cli
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash

Resources: 
  trailMaster: 
    Type: "AWS::CloudTrail::Trail"
    Properties: 
      # TBD: How to specify the name of the trail?
      S3BucketName: !Ref trailBucketName
      IncludeGlobalServiceEvents: true
      IsLogging: true
      IsMultiRegionTrail: true
      EnableLogFileValidation: true

  adminGroup:
    Type: AWS::IAM::Group
    Properties:
      GroupName: group-admins
      ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AdministratorAccess

  adminUser:
    Type: AWS::IAM::User
    Properties:
      Groups:
      - !Ref adminGroup
      LoginProfile: 
        Password: !Ref userPassword
        PasswordResetRequired: false
      UserName: !Ref userName
      # Force MFA to be enable before most other things can be done
      Policies:
      - PolicyName: !Join ["", [!Ref groupName, "-policy"]]
        PolicyDocument:
          Version: 2012-10-17
          Statement:
          -
            Sid: BlockAnyAccessOtherThanAboveUnlessSignedInWithMFA
            Effect: Deny
            NotAction: "iam:*"
            Resource: "*"
            Condition:
              BoolIfExists:
                aws:MultiFactorAuthPresent: false

  adminCliUser:
    Type: AWS::IAM::User
    Properties:
      Groups:
      - !Ref adminGroup
      UserName: !Ref userCliName
```

The use *AWS::CloudFormation::Interface* is optional and so is the policy to prevent non-IAM access without an MFA on the console user. That policy could be added to the group to force the CLI user to also use MFA, however this can be quite cumbersome.

Update the existing stack with your template.

## Complete user setup

* Install (or update) the [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) to the latest version

```bash
$ pip install --upgrade --user awscli
```

* Switch to **IAM**
* Add a MFA to the console account
* Add a MFA and access key to the cli account.
* Configure AWSCLI with the access key and confirm access

```bash
$ aws configure
$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin-cli",
            "UserId": "<USERID>",
            "Arn": "arn:aws:iam::<ACCOUNTID>:user/admin-cli",
            "CreateDate": "2017-05-12T10:07:15Z"
        },
        {
            "Path": "/",
            "UserName": "admin-console",
            "UserId": "<USERID>",
            "Arn": "arn:aws:iam::<ACCOUNTID>:user/admin-console",
            "CreateDate": "2017-05-12T10:07:15Z"
        }
    ]
}
```

## Lockdown account

* Set password policy using the CLI:

```bash
$ aws iam update-account-password-policy --minimum-password-length 12 --require-symbols --require-numbers --require-uppercase-characters --require-lowercase-characters --max-password-age 90 --allow-users-to-change-password --password-reuse-prevention 12 --no-hard-expiry

$ aws iam get-account-password-policy
{
    "PasswordPolicy": {
        "AllowUsersToChangePassword": true,
        "RequireLowercaseCharacters": true,
        "RequireUppercaseCharacters": true,
        "MinimumPasswordLength": 12,
        "RequireNumbers": true,
        "PasswordReusePrevention": 12,
        "HardExpiry": false,
        "RequireSymbols": true,
        "MaxPasswordAge": 90,
        "ExpirePasswords": true
    }
}
```

* Use the conole to set security questions and disable unused regions

# References

[AWS::CloudTrail::Trail](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudtrail-trail.html)

[AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

[Authenticate CLI with MFA](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/)

# Possible Enhancements
* Add MFA requirement to Admin group so cli user also needs to provice MFA, as per [Authenticate CLI with MFA](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/).