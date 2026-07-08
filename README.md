# Lab: IAM Users, Groups, and Policies

## Overview

In this lab, you'll set up IAM for a fictional development team. You'll create users, organize them into groups, assign permissions, and verify that access control works as expected. This simulates real-world IAM administration for cloud teams.

**Estimated Time:** 45-60 minutes (core), up to 90 minutes with bonus tasks

## Learning Objectives

By the end of this lab, you will be able to:

- Create IAM users with appropriate permissions
- Organize users into groups with assigned policies
- Test permissions by signing in as different users
- Enable MFA for enhanced security
- Use IAM Access Analyzer to review permissions
- Apply the principle of least privilege

## Prerequisites

- AWS account with administrative access
- Understanding of IAM concepts from lessons
- Ability to use multiple browser profiles or incognito windows

---

## Scenario

You're the cloud administrator for a development team with the following members:

- **Alice** - Senior Developer (needs full S3 access, read-only EC2)
- **Bob** - Junior Developer (needs limited S3 access to dev bucket only)
- **Charlie** - DevOps Engineer (needs full access to EC2 and S3)

You need to set up IAM users and groups to manage their permissions efficiently.

---

## Exercise 1: Create IAM Groups

Groups make it easier to manage permissions for multiple users.

### Part A: Create Developers Group

**1. Navigate to IAM**

- AWS Console → Search "IAM" → IAM Dashboard

**2. Create Group**

- In left sidebar: User groups → Create group
- Group name: `Developers`
- Don't attach policies yet → Create group

### Part B: Create DevOps Group

**1. Create Second Group**

- User groups → Create group
- Group name: `DevOps`
- Don't attach policies yet → Create group

### Deliverable:

- Screenshot of both groups created: `screenshots/groups-created.png`

---

## Exercise 2: Attach Policies to Groups

Now we'll assign appropriate permissions to each group.

### Part A: Developers Group Permissions

**1. Select Developers Group**

- Click on "Developers" group

**2. Attach Policies**

- Permissions tab → Add permissions → Attach policies
- Search and select:
  - ☑️ `AmazonS3FullAccess`
  - ☑️ `AmazonEC2ReadOnlyAccess`
- Click "Attach policies"

### Part B: DevOps Group Permissions

**1. Select DevOps Group**

- Click on "DevOps" group

**2. Attach Policies**

- Permissions tab → Add permissions → Attach policies
- Search and select:
  - ☑️ `AmazonS3FullAccess`
  - ☑️ `AmazonEC2FullAccess`
- Click "Attach policies"

### Deliverables:

- Screenshot of Developers group permissions: `screenshots/developers-permissions.png`
- Screenshot of DevOps group permissions: `screenshots/devops-permissions.png`

---

## Exercise 3: Create IAM Users

Create three IAM users for your team members.

### Create Alice (Senior Developer)

**1. Create User**

- IAM → Users → Create user
- User name: `alice`
- ☑️ "Provide user access to the AWS Management Console"
- Console access: IAM user
- ☑️ "Users must create a new password at next sign-in"
- Click "Next"

**2. Add to Group**

- ☑️ Add user to group: `Developers`
- Click "Next"

**3. Review and Create**

- Review details → Create user
- **IMPORTANT:** Save the console sign-in URL and password!

### Create Bob (Junior Developer)

**1. Create User**

- Users → Create user
- User name: `bob`
- ☑️ "Provide user access to the AWS Management Console"
- Console access: IAM user
- ☑️ "Users must create a new password at next sign-in"
- Click "Next"

**2. Add to Group**

- ☑️ Add user to group: `Developers`
- Click "Next"

**3. Create**

- Review → Create user
- **Save credentials**

### Create Charlie (DevOps Engineer)

**1. Create User**

- Users → Create user
- User name: `charlie`
- ☑️ "Provide user access to the AWS Management Console"
- Console access: IAM user
- ☑️ "Users must create a new password at next sign-in"
- Click "Next"

**2. Add to Group**

- ☑️ Add user to group: `DevOps`
- Click "Next"

**3. Create**

- Review → Create user
- **Save credentials**

### Deliverables:

- Screenshot of all users created: `screenshots/users-created.png`
- Save credentials in `credentials.txt` (add to .gitignore!)

---

## Exercise 4: Test Permissions

Verify that each user has appropriate access.

### Test Alice's Access

**1. Sign in as Alice**

- Open incognito/private browser window
- Use console sign-in URL for Alice
- Enter credentials and change password

**2. Test S3 Access**

- Navigate to S3
- Try to create a bucket → Should succeed ✅
- Screenshot: `screenshots/alice-s3-access.png`

**3. Test EC2 Access**

- Navigate to EC2
- Try to view instances → Should succeed ✅
- Try to launch instance → Should fail ❌
- Screenshot: `screenshots/alice-ec2-readonly.png`

**4. Sign Out**

### Test Bob's Access

**1. Sign in as Bob**

- New incognito window
- Sign in as Bob and change password

**2. Test S3 Access**

- Navigate to S3
- Try to create bucket → Should succeed ✅
- Screenshot: `screenshots/bob-s3-access.png`

**3. Test EC2 Access**

- Navigate to EC2
- Try to launch instance → Should fail ❌
- Screenshot: `screenshots/bob-ec2-denied.png`

**4. Sign Out**

### Test Charlie's Access

**1. Sign in as Charlie**

- New incognito window
- Sign in as Charlie and change password

**2. Test Full Access**

- S3: Create bucket → Should succeed ✅
- EC2: Try to launch instance → Should succeed ✅
- Screenshot: `screenshots/charlie-full-access.png`

**3. Sign Out**

### Deliverables:

- All permission test screenshots
- Document results in `SOLUTION.md`

---

## Exercise 5: Create Custom Policy

Create a custom policy that allows Bob to access only the `dev-bucket`.

### Part A: Create the Policy

**1. Navigate to Policies**

- IAM → Policies → Create policy

**2. Use JSON Editor**

- Click "JSON" tab
- Replace with:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        },
        {
            "Sid": "DevBucketAccess",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::dev-bucket",
                "arn:aws:s3:::dev-bucket/*"
            ]
        }
    ]
}
```

**3. Name and Create**

- Click "Next"
- Policy name: `DevBucketAccessPolicy`
- Description: "Allows access only to dev-bucket"
- Click "Create policy"

### Part B: Attach to Bob

**1. Navigate to Bob's User**

- IAM → Users → bob

**2. Remove Full S3 Access**

- Permissions tab → Find `AmazonS3FullAccess`
- Remove

**3. Attach Custom Policy**

- Add permissions → Attach policies directly
- Search for `DevBucketAccessPolicy`
- Select and attach

### Part C: Test Custom Policy

**1. Create dev-bucket**

- Sign in as admin
- Create S3 bucket named `dev-bucket`

**2. Sign in as Bob**

- Try to access `dev-bucket` → Should succeed ✅
- Try to access other buckets → Should fail ❌

### Deliverables:

- Screenshot of custom policy: `screenshots/custom-policy.png`
- Screenshot of Bob's restricted access: `screenshots/bob-custom-policy-test.png`

---

## Exercise 6: Enable MFA for Admin User

Add an extra layer of security.

### Steps:

**1. Select User**

- IAM → Users → Select your admin user (or alice)

**2. Enable MFA**

- Security credentials tab
- Multi-factor authentication → Assign MFA device
- Choose "Virtual MFA device"

**3. Set Up Authenticator**

- Scan QR code with authenticator app
- Enter two consecutive codes
- Assign MFA

### Deliverable:

- Screenshot of MFA enabled: `screenshots/mfa-enabled.png`

---

## Bonus Challenges: NA

### Challenge 1: Password Policy

Create a strong password policy:

- IAM → Account settings → Edit password policy
- Minimum length: 12 characters
- Require uppercase, lowercase, numbers, symbols
- Password expiration: 90 days

**Screenshot:** `screenshots/password-policy.png`

---

### Challenge 2: Access Analyzer

**1. Enable IAM Access Analyzer**

- IAM → Access analyzer → Create analyzer
- Analyzer name: `my-analyzer`
- Create analyzer

**2. Review Findings**

- Check for any public or cross-account access
- Screenshot: `screenshots/access-analyzer.png`

---

### Challenge 3: Access Keys for CLI

**1. Create Access Key for Alice**

- IAM → Users → alice
- Security credentials → Create access key
- Use case: "Command Line Interface (CLI)"
- Create key

**2. Configure AWS CLI**

```bash
aws configure --profile alice
# Enter Alice's access key ID and secret
```

**3. Test**

```bash
aws s3 ls --profile alice
```

**Screenshot:** Terminal output

---

## Submission

### Required Deliverables:

1. **Screenshots folder** with all required screenshots
2. **SOLUTION.md** completed with:
   - All group and user configurations
   - Permission test results
   - Custom policy JSON
   - Answers to reflection questions
3. **iam-policy.json** - Your custom policy

### How to Submit:

```bash
git add .
git commit -m "Complete IAM basics lab"
git push origin main
```

Create Pull Request and submit URL to Student Portal.

---

## Success Criteria

- ✅ All 3 IAM users created successfully
- ✅ Users organized into appropriate groups
- ✅ Developers can access S3 but cannot terminate EC2 instances
- ✅ DevOps user has full access
- ✅ Custom policy restricts Bob to dev-bucket only
- ✅ MFA enabled and tested for at least one user
- ✅ Screenshots show successful permission tests
- ✅ All reflection questions answered thoughtfully
- ✅ Code committed and pushed to GitHub
- ✅ Pull request created

---

## Troubleshooting

**Can't create IAM users:**

- Ensure you're signed in as root or admin user
- Check IAM permissions

**"Not authorized" error:**

- This might be expected for limited users
- Document what works and doesn't

**MFA not working:**

- Check device time sync
- Try rescanning QR code

---

## Reflection Questions

1. **Why use groups instead of attaching policies directly to users?**
2. **What are the risks of giving everyone AdministratorAccess?**
3. **How would you organize IAM for 50 developers across 5 projects?**
4. **What happens if you delete an IAM user?**

Answer in `SOLUTION.md`.

---

**Good luck! 🔐**
