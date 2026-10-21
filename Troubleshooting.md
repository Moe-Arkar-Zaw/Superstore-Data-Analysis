# AWS Troubleshooting Log

## 1. Finding the AWS account ID

**Environment:** AWS Management Console (old and new layouts)
**Problem:** Could not find the 12-digit account ID in the new console.
**Cause:** The new console layout no longer shows the account ID in the account menu.

**Solution:**
- Old console: click your account name (top right) and copy the Account ID from the profile section.
- New console: read it from the URL bar. The 12-digit number at the start of the address is the account ID
  (e.g. `123456789012-xxxxxxxx.region.console.aws.amazon.com`).

**Alternative:** IAM dashboard (Account ID on the right), or run in CloudShell:

```bash
aws sts get-caller-identity --query Account --output text
```

**Notes:** The account ID is not a secret, but avoid sharing it publicly with other account details.

---

## 2. Cannot set a console password for an IAM user

**Environment:** AWS account created with "Sign up for AWS" (new) using a Google/GitHub/Apple login (Builder ID), Free plan
**Problem:** IAM user was created, but there is no option to give it a console password.
The "Provide user access to the AWS Management Console" checkbox is missing on the Create user page,
and the Security credentials tab has no Console sign-in section.

**Error message** (from CloudShell, `aws iam create-login-profile`):
`AccessDenied ... is not authorized to perform: iam:CreateLoginProfile ... with an explicit deny in a service control policy`

**Cause:** Accounts created through the new sign-up sit inside an AWS-managed organization
with service control policies that block console passwords for IAM users. An explicit SCP deny
overrides even full admin permissions.

**Solution:** Create a standard account:
1. Go to the AWS sign-up page and choose **Sign up (advanced)**.
2. Use a different root email address from the first account.
3. Sign in as the root user, then create IAM users with console access from IAM.

**Workaround:** For learning, keep using the existing account. Console and CloudShell already have full access,
so an IAM user isn't required.

**Notes:**
- The standard account may require a credit card and may not receive the free credits. Set a billing budget alert.
- Never paste passwords or access keys into notes or screenshots.

--
## 3. IAM user gets AccessDenied on AWS Glue

**Environment:** Standard AWS account (created with Sign up advanced), IAM user `moe`, AWS Glue console, region ap-southeast-2

**Problem:** Opening Glue as the IAM user failed with:
`AccessDeniedException: User ... is not authorized to perform: glue:GetDatabases ... because no identity-based policy allows the glue:GetDatabases action`

**Error message** `AccessDeniedException (status: 400): User: arn:aws:iam::733347225020:user/moe is not authorized to perform: glue:GetDatabases on resource: arn:aws:glue:ap-southeast-2:733347225020:catalog because no identity-based policy allows the glue:GetDatabases action`

**Cause:** A new IAM user has no permissions by default. The user's Permissions tab showed 0 policies, so every service action was denied.

**Solution:** Sign in as the root user (or an admin), then:
1. Go to IAM > Users > moe > Permissions.
2. Choose Add permissions > Attach policies directly.
3. Tick `AdministratorAccess` (or, for limited access, `AWSGlueConsoleFullAccess`, `AmazonS3FullAccess` and `AmazonAthenaFullAccess`).
4. Click Next > Add permissions, then refresh the Glue console and retry.

**Note:** The user cannot grant permissions to itself, so this has to be done from root or another admin. Policies can take up to about 30 seconds to apply. Enable MFA on any user with administrator access, and keep root for billing and account-level tasks only.