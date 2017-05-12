# EC2 Web Application - Lab 010 - Master Account
By David Chatterton

## Goal

The course creates other accounts and only uses the master account for setting up organisations and new accounts, IAM users to access new accounts and billing.

Lab 01 is to create and lock down the master account if this has not been done previously.

Otherwise jump to the next lab if this is already done

## Prereqs
* Nothing

## Completed State
* A master account has been created and locked down
  * Define prefix for global names
  * Create email address
  * Create AWS account
  * Enable CloudTrail
    * Monitoring all regions
    * File validation on
  * 2FA on root account
  * Password policy defined
  * Disable all other regions
  * Set security questions
  * Configure billing

# Master Account

**Principle**: Secure and minimise use of the root account.

## Global Naming Prefix

Consider using a common prefix that you use for *global* names, including
* accounts
* email addresses, and
* S3 buckets

created in this course. In a real production environment this is likely to be the company name.

The examples will use `cle2wa` but you may need to select something different to avoid name clashes. If you do select another prefix you will need to substitute that prefix wherever you see `cle2wa`.

## Create Account

* Each account requires a unique email address, so if you don't want to use your current email address you can easily create a new address with [gmail](https://accounts.google.com/SignUp?hl=en)
  * *Tip* - Within an organisation it is a good idea to setup an email alias (that goes to a few people) for the account rather than using an individuals email account who could leave the company
  * *Tip* - It's a good idea to secure this gmail account by setting up 2-factor
  * *Tip* - If using Chrome its also easy to [switch between gmail accounts](https://support.google.com/accounts/answer/1721977?hl=en), especially useful if you want to log into multiple AWS consoles with different accounts at the same time.

* Now follow the steps documented in [Create AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

* *Tip* - Immediately after the account is created CloudTrail can be configured for the account. If you want to capture everything you do to the account from this point on, you can jump to [lab-02](../lab-02-cloudtrail) now and then come back and complete the rest of this lab.

## CloudTrail

I recommend setting up CloudTrail immediately so that you capture everything you do from this point on. This is especially handy if something does not work and you don't understand why you can at least see the API calls being made.

*Note*: [Lab 015](../lab-015-cloudtrail) may replace this section with a cloudtrail configuration that further restricts access to the cloudtrail logs. 

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
* Select **Advanced**
  * Select **Encrypt log files** and create a KMS key called *cle2wa-key-trail-master*
    * As more accounts and trails are created we will need to refer to this key.
  * Check that **Enable log file validation** is on
* Press **Create**

## Secure Root Account

Adding a second factor to the root account is one of the best way to secure access to this account. 

* Select the `user@account` drop down
* Select **My Security Credentials**
* Select **Dashboard**
* Select **Activate MFA on your root account** and go through the process to generate a virtual MFA for the root account

## Configuring a CLI user

This course assumes that a CLI user is not configured for the root account and only the console is (rarely) used.

If a CLI user was to be configured then the policy attached to the role should require [2FA on support API calls](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html).

**_Do not give the root account CLI access, always create a new user for CLI access to the master account_**. Limit root access to a password and 2FA and do not also add the burden of protecting the access keys.

## Setup Password Policy

The default password policy defined in new accounts is not very strong and should be updated.

* Select **Account settings**
* I recommend you select all options
  * The only option I may not select in non-master accounts is the requirement for the administrator to reset an expired account password
* Because I use a password manager I typically set parameters higher:
  * minimum password length to 12, 
  * password expiry to 90 days, and
  * number of passwords to remember to 12 

## Disable other regions

The master account should ideally be used for managing organizations and billing and nothing else. Therefore no other regions are required and they should be disabled.

* While still in the **Account Settings** area, deactive all regions

## Setup Account Information

For a corporate account, all the billing information should be configured.

* Select the `user@account` drop down
* Select **My Account**
* Complete **Configure Security Challenge Questions** section as this is the best way to recover your account if you lose your 2FA for the account.

## Setup Billing Information

* Select the `user@account` drop down
* Select **My Billing Dashboard**
* Select **Preferences** and ensure **Receive Billing Alerts** is enabled
* Select **Budgets**
* Create a [budget](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/budgets-managing-costs.html) and alarm so you are alerted if you leave something running that costs more than you expect.


# References

[Create AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

[AWS CloudTrail](https://aws.amazon.com/cloudtrail/)

[IAM Best Practices](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

[Password Policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html)

[2FA for API access](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html)

[Budgets](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/budgets-managing-costs.html)

# Possible Enhancements
* If this was the master account for your business, consider what additional steps you could take to secure the root user of this account
  * The security questions are only needed if other methods for accessing the account fail. The answers to these questions don't have to make sense, they just need to be secure, so make up some answers, record them and store the answers separately to the MFA tokens. For example, give the answers to your legal counsel and advise them on the circumstances that you would require them.
  * Purchase [hardware tokens](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_physical.html) and secure the token in a safe that very few people have access to. 

* Encrypt the cloud trail logs by setting up a KMS key in the same region as the cloud trail bucket.

* Setup CloudWatch to consume the CloudTrail trail and generate events.

* Whatever steps you take, document this as a company policy so that staff know what to do in normal and emergency situations.
