# EC2 Web Application - Lab 030 - Automating Cross Account Setup

Now that we understand the steps to setup a group and user to access a new
account in [lab-025](../lab-025-manual-cross-account), we can
automate these sames steps through cloud formation so we don't have to use
the console to do this again.
* When a new subaccount is created in the
future we can run this stack again to create a new group and user for
that account with the confidence that it will work and meet our
requirements.
* If those requirements change in future, thats OK, we know
we have a single place to change and then we can rerun that stack on
all those accounts to apply those changes.

## Goal
* Develop and test cloudformation scripts to automatically provision
the group, user and policies to be able to switch roles into a new account.
* Because this is automated we can make further improvements on what is
deployed to define what this user can and cannot do in the master account. These steps would become tedious through the console if you
had to do this again and again. 

## Prereqs
* Master account is created and locked down
* An organisation has been created for the master account
* A subaccount has been created in the organisation, but no group, user or
policies are required in the master account.
  * If this has already been down in [lab-020](../lab-020-organization) you can build this using different names so you can compare, and then remove
  the IAM resources you created by hand in that lab.
* Assumes you have some CloudFormation experience. If not I suggest you do the [Advanced AWS CloudFormation](https://acloud.guru/course/aws-advanced-cloudformation/dashboard) course on [acloud.guru](https://acloud.guru).

## Completed State
* IAM user setup im master account and can assume role in new account
mostly using cloud formation

# Lab 030 - Automated Cross Account Setup

When developing cloudformation configurations you will quickly find the tools you might be used to with other programming languages to help you debug problems just don't exist. Therefore it pays to be incremental with the script and not try to build everything at once, especially since you can easily update the stack with the latest version to check it still works.

Ideally try to implement these scripts yourself and then refer to my scripts for reference. If your script is not exactly the same that is OK, yours might be better, but look and understand the differences.

**Principle**: Secure and minimise use of the root account.

**Principle**: Segregate roles and responsibilities.

**Principle**: Automate steps that are likely to repeated to ensure they
are repeatable and consistent

## Create Group

* To create the group we are going to need a name and the subaccount ID, so first define the two parameters *accountId* and *groupName*.
* Ideally specify constraints so we know the input data is valid.
* Next define the group resource, defining properties for the *GroupName* and *Policies*.
* The policy will require a *PolicyName* and *PolicyDocument* that contains the policy from [lab-025](../lab-025-manual-cross-account). My examples are in YAML since that is easier to read, but that means you have to convert thge policy document to YAML too.
* Lastly specify the ARN of the newly created group as an output

You script should look something like [this](scripts/cf-subaccount-access-1.yaml):

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: Stack for creating group and user to switch roles into a subaccount. Add MFA to new user once stack build is complete.

Parameters:
  accountId:
    Type: String 
    MinLength: 12
    MaxLength: 12
    AllowedPattern: ^[0-9]*$
    ConstraintDescription: 12 digits
  groupName:
    Type: String
    MinLength: 3
    MaxLength: 64
    AllowedPattern: ^[A-Za-z0-9][A-Za-z0-9\-]*[A-Za-z0-9]$
    ConstraintDescription: 3 to 64 upper case, lower case or dashes, must not start or end with a dash

Resources:
  groupAccount:
    Type: AWS::IAM::Group
    Properties:
      GroupName: !Ref groupName
      Policies:
      # Assume Role to switch to subaccount
      - PolicyName: !Join ["", [!Ref groupName, "-policy"]]
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action:
            - sts:AssumeRole
            Condition:
              Bool:
                aws:MultiFactorAuthPresent: "true"
            Resource: !Join [ "", ["arn:aws:iam::", !Ref accountId, ":role/OrganizationAccountAccessRole"]]

Outputs:
  groupArn:
    Description: Group ARN
    Value: !GetAtt groupAccount.Arn
```

* Navigate to **CloudFormation**
* Select **Create Stack**
* Use the **Upload a template to S3** and **Choose File**
* Select your script file
* Provide a
  * *Stack Name* (for example *stack-switch-account-ACCOUNTID and substitute the account id, or stack-switch-dev-account),
  * the subaccount *Account ID*, and
  * the *Group Name* (for example group-acount-ACCOUNTID or group-dev-account)
* Press **Next** and **Next**
* Tick the **I acknowledge that AWS CloudFormation might create IAM resources with custom names** box 
* Press **Create**
* Fix any errors if your stack was rolled back and try again.
* Once successful, check that a group with the correct incline policy has been created.
  * When reviewing the inline policy it is easier to read if you press **Edit Policy**.
  * It is possible for the stack creation to succeed but the policy to have errors, so ensure **Validate Policy** returns no errors.

## Create User

* Add to your cloudformation template the resource definition for the [user](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-iam-user.html)
* You will need to add parameters for the *User Name* and *Password*
* Ensure the password is hidden and the user is added to the group

The user resource should look something like [this](scripts/cf-subaccount-access-2.yaml):
```yaml
  userAccount:
    Type: AWS::IAM::User
    Properties:
      Groups:
      - !Ref groupAccount
      LoginProfile: 
        Password: !Ref userPassword
        PasswordResetRequired: false
      UserName: !Ref userName
```

* In **CloudFormation** select the stack from the previous step
* Select **Actions** and **Update Stack**
* Use the **Upload a template to S3** and **Choose File** and select your updated file
* Click through the same selections as before, specifying the user name and password.
* Once complete check the new user has been created and assigned to the group

With the group, user and group policy created, you can try to logout, login as new user and switch roles. However you will encounter some problems. 

The first issue is the URL to switch roles can be found in the [documentation](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-cross-account-role) but it would be nice to generate the URL for them with all the fields filled out.

## Improve Parameters

We can make the interface for entering parameters more user friendly by specfying [AWS::CloudFormation::Interface](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudformation-interface.html) within the *Metadata* section of the document.

* Add a *ParameterGroups* definition for the two parameters
* Add a *ParameterLabels* definition to improve the labels

You script should include something like [this](scripts/cf-subaccount-access-3.yaml):

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      -
        Label:
          default: CrowdLabs EC2 Web Application Subaccount Switch Roles
        Parameters:
          - accountId
          - groupName
          - userName
          - userPassword
    ParameterLabels:
      accountId:
        default: Sub-account ID
      groupName:
        default: Group Name
      userName:
        default: User Name
      userPassword:
        default: User Password
```

If you try to update the stack with this change you will get an error as the updated configuration file does not make any changes to any resources. 

Another change that can be make to make the stack more user friendly is to generate as an output the URL the user can use to switch role, at least the first time.

The URL can be found [here](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-cross-account-role) and your addition should something like this:

```yaml
  switchRoleURL:
    Description: URL to switch role to access sub-account
    Value: !Join [ "", ["https://signin.aws.amazon.com/switchrole?account=", 
                        !Ref accountId,
                        "&roleName=OrganizationAccountAccessRole&displayName=",
                        !Ref userName,
                        "@",
                        !Ref accountId]]
```

* In **CloudFormation** select the stack from the previous step
* Select **Actions** and **Update Stack**
* Use the **Upload a template to S3** and **Choose File** and select your updated file
* Click through the same selections as before, the paramters can stay the same but should now be in a more user friendly format. 
* When the update is complete check the stack outputs to see the URL.

## Create Policy

Now that the URL is generated as an output, they can navigate to the stack to see and click on it, right? No they can't, that have no permissions in the master account to do anything except assume the role in the subaccount.

The simplest change is to give the user read-only access to CloudFormation, since they should not be able to perform any cloudformation actions.

* Navigate to **IAM** and thne **Policies**
* Find the managed read-only policy for cloduformation and select the name
* At the top of the screen you will see the ARN for the policy. Copy this.
* Add a *ManagedPolicyArns* to the group:

```yaml
      ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AWSCloudFormationReadOnlyAccess
```

The next problem with not having permissions in the master account is the user does not have the ability to configure MFA for themselves. Since this could be another person using this user to access this account, we can't do this for them.

Amazon have provided a [document](http://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html) on how you can do this. So we need to add this policy to the user since the user should only be able to update their login and not someone else in their group.

The documentation from Amazon specifies this as a JSON policy document, so here it is in YAML:

```yaml
  userIAMPolicy:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: !Join [ "", [!Ref userName, "-policy-user-iam-access"]]
      Users:
      - !Ref userName
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Sid: AllowAllUsersToListAccounts
            Effect: Allow
            Action:
            - iam:ListAccountAliases
            - iam:ListUsers
            - iam:GetAccountSummary
            Resource: "*"
          - 
            Sid: AllowIndividualUserToSeeAndManageTheirOwnAccountInformation
            Effect: Allow
            Action:
            - iam:ChangePassword
            - iam:CreateAccessKey
            - iam:CreateLoginProfile
            - iam:DeleteAccessKey
            - iam:DeleteLoginProfile
            - iam:GetAccountPasswordPolicy
            - iam:GetLoginProfile
            - iam:ListAccessKeys
            - iam:UpdateAccessKey
            - iam:UpdateLoginProfile
            - iam:ListSigningCertificates
            - iam:DeleteSigningCertificate
            - iam:UpdateSigningCertificate
            - iam:UploadSigningCertificate
            - iam:ListSSHPublicKeys
            - iam:GetSSHPublicKey
            - iam:DeleteSSHPublicKey
            - iam:UpdateSSHPublicKey
            - iam:UploadSSHPublicKey
            - iam:ListUserPolicies
            - iam:ListAttachedUserPolicies
            - iam:ListGroupsForUser
            Resource: 
            - !GetAtt userAccount.Arn
          -
            Sid: AllowIndividualUserToListTheirGroups
            Effect: Allow
            Action:
            - iam:ListGroups
            Resource:
            - !Join ["", ["arn:aws:iam::", !Ref "AWS::AccountId", ":group/*"]]
          -
            Sid: AllowIndividualUserToSeeTheirGroupPolicies
            Effect: Allow
            Action:
            - iam:ListGroupPolicies
            - iam:ListAttachedGroupPolicies
            Resource: !GetAtt groupAccount.Arn
          -
            Sid: AllowIndividualUserToListTheirOwnMFA
            Effect: Allow
            Action:
            - iam:ListVirtualMFADevices
            - iam:ListMFADevices
            Resource:
            - !Join ["", ["arn:aws:iam::", !Ref "AWS::AccountId", ":mfa/*"]]
            - !GetAtt userAccount.Arn
          -
            Sid: AllowIndividualUserToManageTheirOwnMFA
            Effect: Allow
            Action:
            - iam:CreateVirtualMFADevice
            - iam:DeactivateMFADevice
            - iam:DeleteVirtualMFADevice
            - iam:RequestSmsMfaRegistration
            - iam:FinalizeSmsMfaRegistration
            - iam:EnableMFADevice
            - iam:ResyncMFADevice
            Resource:
            - !Join ["", ["arn:aws:iam::", !Ref "AWS::AccountId", ":mfa/", !Ref userName]]
            - !GetAtt userAccount.Arn
          -
            Sid: BlockAnyAccessOtherThanAboveUnlessSignedInWithMFA
            Effect: Deny
            NotAction: "iam:*"
            Resource: "*"
            Condition:
              BoolIfExists:
                aws:MultiFactorAuthPresent: false
```

*Note*: At the time of writing, the AWS doc did not include the following actions which you may also grant a user:
- iam:ListUserPolicies
- iam:ListAttachedUserPolicies
- iam:ListGroupsForUser

and for all groups in the account:
- iam:ListGroups

and for the specific group created here:
- iam:ListGroupPolicies
- iam:ListAttachedGroupPolicies

I have added these actions to the policy. The complete CloudFormation document can be found [here](scripts/cf-subaccount-access-4.yaml) to review.

Update your stack once again with these changes and you should now be able to login as the new user, create an MFA, click the generated URL to switch roles and access the new account.


# References

[CloudFomration Best Practices](https://www.slideshare.net/AmazonWebServices/aws-cloudformation-best-practices)

[Advanced AWS CloudFormation Course](https://acloud.guru/course/aws-advanced-cloudformation/dashboard) 

[Organization Cross-Account Access](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-cross-account-role)

[CloudFormation AWS::IAM::Group](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-iam-group.html)

[AWS::CloudFormation::Interface](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudformation-interface.html)

[CloudFormation AWS::IAM::User](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-iam-user.html)

[CloudFormation AWS::IAM::Policy](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-policy.html)

[Users self-manage login](http://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html)

# Possible Enhancements
* Create the subaccount using cloudformation
* Instead of prompting for the user password here, force the password to be reset on first login
* Email the account details to the user so they don't need to have cloudformation read access in the master account