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

### 3. Set up Production Environment

1. Go to your GitHub repository settings → Environments
2. Create a new environment called `production`
3. Add protection rules:
   - ✅ Required reviewers (add yourself or team members)
   - Optionally set wait timer if desired

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
            "Action": [
                "s3:ListBucket"
            ],
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

1. Go to your repository's Actions tab
2. Click on the running workflow
3. You'll see a "Review deployments" button for the production environment
4. Click it and approve the deployment to production

## File Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-s3.yml    # GitHub Actions workflow
├── index.html              # HTML file to deploy
└── README.md              # This file
```
