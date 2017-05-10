# EC2 Web Application - Lab 020 - Organisations

## Goal

Establish a separate account from the master account for the development environment. 

*Tips*: It is a good idea for any experimental environment, including while doing a course, to use a separate account. This prevents other accounts from being contaminated and at the end of the course it is relatively easy to delete the account. It also allows you to take advantage of the free tier again.

## Prereqs
* Master account is in place and locked down

## Completed State
* Dev organisation and account is in place

* IAM user setup im master account and can assume role in new account

# Organisations

**Principle**: Secure and minimise use of the root account.

**Principle**: Segregate roles and responsibilities.

## Create New Email Address

* Generate a new email account for this new account
  * [gmail](https://accounts.google.com/SignUp?hl=en)

## Create Organization

Create an Organization for the master account.

* From the **user@account** dropdown select **My Organization**
* Select **Create Organization**
* Select **Enable All Features** so that we can create Organization Units and Policies
* Select **Create Organization**

## Create Organization Unit

Organization units are helpful if different policies apply to a group of accounts. In this case any accounts for development purposes might have different set of policies to accounts used for production workloads.

* Under the *Accounts* tab take note of the master account's ID for later in this lab
* Select **Organize accounts**
* Select the Root box
* Select **+New orgnaizational unit**
* Enter a name for the organizational unit, I went with *ou-dev*

## Create Account

Create a new account using the email address generated earlier and move it into the *ou-dev* organization unit.

* Select the **Accounts** tab
* Select **Add Account**
* Select **Create Account**
* Enter an account name and the email address you generated earlier
* Select **Create**
* Once completed record the account ID of the new account
* Select the **Organizae accounts** tab
* Select the new account and **Move**
* Select the dev organization unit and **Move**

# References

[Create new account within your organization](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_create.html#orgs_manage_accounts_create-new)

# Possible Enhancements
* An organisation policy defined