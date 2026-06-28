# 🚀 Portfolio Deployment using GitHub Actions 

This project uses GitHub Actions to automatically deploy the portfolio website to an Amazon S3 bucket every time new changes are pushed to the `main` branch.

## 📌 Overview

- CI/CD pipeline powered by GitHub Actions

- Automatic deployment to an AWS S3 static website hosting bucket

- AWS IAM User configured with AdministratorAccess for full permissions

- Deployment triggered on every push to the main branch

## ⚙️ GitHub Actions Workflow

- This repository includes a deployment workflow in:
` .github/workflows/main.yml`

```bash
name: Portfolio Deployment

on:
  push:
    branches:
    - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v1

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Deploy static site to S3 bucket
      run: aws s3 sync . s3://Your_Bucket_Name --delete
```

## 🔐 AWS Setup

### IAM User
#### Create a new IAM user and attache Policy: `AdministratorAccess`
This grants full permissions to manage S3 and any other AWS services.
The user’s credentials are stored as GitHub Secrets:

- AWS_ACCESS_KEY_ID

- AWS_SECRET_ACCESS_KEY

## Here is an example
![Secret Key Added](https://github.com/Mainul41561/portfolio-mainul/blob/main/githubsecret.png)


## 🪣 S3 Bucket Configuration
### Your S3 bucket is configured for Static Website Hosting:

- 1.Bucket Name: Your_Bucket_Name

- 2.Static Website Hosting enabled

- 3.Public access allowed

- 4.Bucket Policy for public access:
```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::protfolio-mainul/*"
    }
  ]
}
```
## 🚀 Automatic Deployment Process

#### Once configured:

- Push changes to main

- GitHub Actions triggers the workflow

- Files are synced to the S3 bucket

Your portfolio will update automatically
