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

## Create IAM Group for accessing new account

Create a group with an inline policy to access the new account and add a user to thge group.

* Navigate to **IAM**
* Create a group called *group-dev-account* with no policies attached
* Select the new group, go to the **Permissions** tab
* Select the *Inline Policies* drop down arrow and create an inline policy
  * Set **Effect** to **Allow**
  * Select **AWS Service** as **AWS Security Token Service**
  * Select **Assume Role** as the only **Action**
  * Insert this ARN `arn:aws:iam::NEWACCOUNTID:role/OrganizationAccountAccessRole`, replacing *NEWACCOUNTID* withthe account ID of the new account.
  * Add a condition to enfore MFA
    * Set **Condition** to **Bool**
    * Set **Key** to **aws:MultiFactorAuthPresent**
    * Set **Value** to**true**
    * Press **Add Condition**
  * Press **Add Statement** and **Next Step**
  * The policy statement should look like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1494246751000",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            },
            "Resource": [
                "arn:aws:iam::NEWACCOUNTID:role/OrganizationAccountAccessRole"
            ]
        }
    ]
}
```

* Press **Apply Policy**

## Create a user to access the new account

Use a new user and not the root user to access sub accounts.

* Create a user *cle2wa-dev* and the user to *group-dev-account*.
* Setup MFA for user *cle2wa-dev*
* Optionally change the **IAM signin link** for the master account

## Access the new account

We can now use the new user to switch roles into the new account.

* Logout as root
* Log back in as *cle2wa-dev* in the master account
* Open the *cle2wa-dev@cle2wa* drop down and select **Switch role**
* Enter the new account ID for **Account**
* Enter *OrganizationAccountAccessRole* for **Role**
* Specify the name to appear in the user dropdown, eg *cle2wa-dev*
* Press **Switch Role**
* You should now be in the new account, you can then select **Back to cle2wa-dev** to get back to the same user but in the master account 

# References

[Organization Cross-Account Access](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-cross-account-role)

# Possible Enhancements
* Now with organisations in place, the purpose of the master account can be limited to managing the organisation under it and billing. Nothing else. The benefit of doing this is you can reduce the need to access the root account.
  * So instead of creating more groups/users in the master account for accessing sub-accounts, create another sub-account and create all the IAM users there.
  * The completxity of this approach is that new accounts default to creating a role that trusts the master account, at this time you cannot specify another account.
  * This means that you have to create a user in the master account, log into the new account to establish a trust relationship to the IAM account.

* AWS CodeStar requires an IAM user and not a user who has assumed a role. I expect this limitation will be removed in a future update removing the need to create a user in the development account.

* An organisation policy defined