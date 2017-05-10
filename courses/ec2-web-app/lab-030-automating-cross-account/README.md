# EC2 Web Application - Lab 030 - Automating Cross Account Setup

Now that we can setup a group and user to access a new account in
[lab-025](../lab-025-manual-cross-account), we can
automate this through cloud formation so we don't have to use the
console to do this again. When a new subaccount is created in the future
we can run this stack again to create a new group and user.

## Goal
* Develop and test cloudformation scripts to automatically provision
the group, user and policies to be able to switch roles into a new account.
* Because this is automated we can make further improvements on what is
deployed to define what this user can and cannot do in the master account.


## Prereqs
* Master account is created and locked down
* An organisation has been created for the master account
* A subaccount has been created in the organisation, but no group, user or
policies are required in the master account.
  * If this has already been down in [lab-020](../lab-020-organization) you can build this using different names so you can compare, and then remove
  the IAM resources you created by hand in that lab.

## Completed State
* IAM user setup im master account and can assume role in new account
mostly using cloud formation

# Lab 030 - Automated Cross Account Setup

**Principle**: Secure and minimise use of the root account.

**Principle**: Segregate roles and responsibilities.

**Principle**: Automate steps that are likely to repeated to ensure they
are repeatable and consistent

## <step name>

# References

[Organization Cross-Account Access](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-cross-account-role)

[Users self-manage login](http://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html)

# Possible Enhancements
* describe further steps or alternatives that could be considered