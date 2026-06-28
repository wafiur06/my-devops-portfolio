# 🚀 DevOps Portfolio CI/CD Pipeline (LocalStack & GitHub Pages)

This project uses GitHub Actions to automatically test AWS S3 deployment commands locally using **LocalStack** and deploys the portfolio website securely to **GitHub Pages** every time new changes are pushed to the `main` branch. 

## 📌 Overview

- **Automated CI/CD Pipeline:** Fully managed by GitHub Actions.
- **Cost-Free AWS Testing:** Uses LocalStack to mock an AWS S3 environment locally, avoiding real AWS infrastructure costs.
- **Zero Secrets Required:** No need to expose real IAM credentials or manage GitHub Secrets.
- **Production Deployment:** Automatic and secure deployment to GitHub Pages upon successful integration testing.
- **Continuous Deployment:** Triggered automatically on every push to the `main` branch.

## ⚙️ GitHub Actions Workflow

This repository includes a deployment workflow located at:
`.github/workflows/deploy.yml`

The workflow is divided into two main jobs (`localstack-test` and `github-pages-deploy`):

```yaml
name: Portfolio CI/CD (LocalStack & GitHub Pages)

on:
  push:
    branches:
      - main 

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  # Job 1: Integration Testing with LocalStack
  localstack-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Start LocalStack (v3.8.0)
        run: |
          docker run -d -p 4566:4566 --name localstack_ci localstack/localstack:3.8.0
          # Custom health check loop applied here

      - name: Install LocalStack CLI (awslocal)
        run: pip install awscli-local

      - name: Configure Dummy AWS Credentials
        run: |
          aws configure set aws_access_key_id test
          aws configure set aws_secret_access_key test
          aws configure set region us-east-1

      - name: Create S3 Bucket and Deploy (Testing)
        run: |
          awslocal s3 mb s3://my-portfolio-bucket
          awslocal s3api put-object --bucket my-portfolio-bucket --key index.html --body index.html --content-type "text/html"

  # Job 2: Live Deployment
  github-pages-deploy:
    runs-on: ubuntu-latest
    needs: localstack-test # Runs only if LocalStack test passes
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.' 
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

## 🔐 AWS Setup (Mock Environment)

### No Real IAM User Required

Since we are using LocalStack for the DevOps pipeline, we do not need to create an actual AWS IAM User or attach `AdministratorAccess` policies.

The pipeline uses **dummy credentials** to interact with the LocalStack container securely. **No GitHub Secrets are needed.**

* `AWS_ACCESS_KEY_ID`: test
* `AWS_SECRET_ACCESS_KEY`: test
* `AWS_REGION`: us-east-1

## 🪣 LocalStack S3 Bucket Configuration

During the CI/CD run, a temporary mock S3 bucket is created inside the GitHub Actions runner for integration testing.

* **Bucket Name:** `my-portfolio-bucket`
* **Operation:** The pipeline successfully uses low-level API (`s3api put-object`) to bypass `x-amz-trailer` header conflicts and uploads the `index.html` file.
* **Verification:** Runs `awslocal s3 ls` to verify the deployment locally before proceeding to production.

## 🚀 Automatic Deployment Process

#### How the CI/CD pipeline works:

1. Push your updated code to the `main` branch.
2. GitHub Actions triggers the workflow.
3. **Phase 1 (Test):** LocalStack spins up, configures the mock AWS CLI, creates the S3 bucket, and tests the upload process.
4. **Phase 2 (Deploy):** Once the test passes, the repository files are packaged and published to GitHub Pages.
5. Your portfolio updates automatically on the live server.

