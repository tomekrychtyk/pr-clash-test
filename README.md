# PR Clash Test - S3 Deployment

This project contains a simple HTML file that gets automatically deployed to S3 buckets using GitHub Actions.

## Deployment Workflow

The deployment process consists of three stages:

1. **Staging Deployment** - Automatically deploys to staging S3 bucket when code is pushed to `main` branch
2. **Manual Approval** - Waits for manual approval before proceeding to production
3. **Production Deployment** - Deploys to production S3 bucket after approval

## Setup Instructions

### 1. Create S3 Buckets

Create two S3 buckets in your AWS account:

- One for staging (e.g., `my-app-staging`)
- One for production (e.g., `my-app-production`)

Make sure both buckets are configured for static website hosting if you want to serve the HTML directly.

### 2. Configure GitHub Secrets

Go to your GitHub repository settings → Secrets and variables → Actions, and add the following secrets:

#### Required Secrets:

- `AWS_ACCESS_KEY_ID` - Your AWS access key ID
- `AWS_SECRET_ACCESS_KEY` - Your AWS secret access key
- `AWS_REGION` - Your AWS region (e.g., `us-east-1`)
- `STAGING_S3_BUCKET` - Name of your staging S3 bucket
- `PRODUCTION_S3_BUCKET` - Name of your production S3 bucket

### 3. Set up Production Environment (CRITICAL for Manual Approval)

This step is **essential** for the manual approval to work:

1. Go to your GitHub repository **Settings** → **Environments**
2. Click **New environment**
3. Name it exactly `production`
4. Configure protection rules:
   - ✅ **Required reviewers** - Add yourself and/or team members who can approve deployments
   - ✅ **Prevent self-review** (optional but recommended)
   - Optionally set **Wait timer** if you want a minimum delay

**Without this setup, the workflow will not pause for approval!**

### 4. AWS IAM Permissions

Make sure your AWS user has the following permissions for both S3 buckets:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-staging-bucket/*",
        "arn:aws:s3:::your-production-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::your-staging-bucket",
        "arn:aws:s3:::your-production-bucket"
      ]
    }
  ]
}
```

## How It Works

1. When you push to the `main` branch, the workflow automatically triggers
2. The HTML file is deployed to the staging S3 bucket
3. The workflow pauses and waits for manual approval in the GitHub Actions interface
4. Once approved, the HTML file is deployed to the production S3 bucket
5. URLs for both deployments are displayed in the workflow logs

## Workflow Triggers

- ✅ Push to `main` branch
- ❌ Pull requests (workflow only runs after merge)

## Manual Approval Process

1. **After staging deployment completes:**

   - Go to your repository's **Actions** tab
   - Click on the running workflow
   - You'll see the workflow is paused at "Wait for Manual Approval"

2. **Review the staging deployment:**

   - The workflow will show a staging URL in the logs
   - Click the URL to review the deployed site
   - Test that everything works as expected

3. **Approve for production:**

   - In the workflow page, you'll see a yellow **"Review deployments"** button
   - Click it to see the approval dialog
   - You can add a comment (optional)
   - Click **"Approve and deploy"** to proceed to production
   - Or click **"Reject"** to stop the deployment

4. **After approval:**
   - The production deployment will automatically start
   - You'll get the production URL in the final step logs

## File Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-s3.yml    # GitHub Actions workflow
├── index.html              # HTML file to deploy
└── README.md              # This file
```
