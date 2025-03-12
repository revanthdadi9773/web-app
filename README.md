# Web App Deployment using GitHub Actions and AWS S3

## Overview
This guide explains the step-by-step process of deploying a static web application to AWS S3 and setting up an automated deployment workflow using GitHub Actions.

## Prerequisites
Before starting, ensure you have the following:
- An **AWS S3 bucket** (for hosting the web app)
- **CloudFront** (optional, for CDN and caching)
- **GitHub repository** (for storing your project code)
- **GitHub Actions** setup (for automation)

---

## Deployment Steps

### 1. **Setting Up the S3 Bucket**
- Create an **S3 bucket** in the AWS console.
- Enable **static website hosting**.
- Set **Bucket Policy** to allow public read access (if needed for public access).
- Update **CORS policy** to allow frontend communication with the API if required.

### 2. **Configuring GitHub Secrets**
- Go to **GitHub Repository > Settings > Secrets**.
- Add the following secrets:
  - `AWS_ACCESS_KEY_ID` → Your AWS Access Key
  - `AWS_SECRET_ACCESS_KEY` → Your AWS Secret Key
  - `AWS_S3_BUCKET` → Your S3 bucket name
  - `AWS_REGION` → Your AWS region

### 3. **GitHub Actions Workflow for Deployment**
Create a `.github/workflows/deploy.yml` file in your project:

```yaml
name: Deploy to S3
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Install AWS CLI
        run: |
          sudo apt update
          sudo apt install -y awscli

      - name: Deploy to S3
        run: |
          aws s3 sync ./build s3://${{ secrets.AWS_S3_BUCKET }} --delete
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
```

### 4. **Configuring CloudFront (Optional but Recommended)**
- Create a **CloudFront distribution** pointing to the S3 bucket.
- Enable **custom domain & SSL** if required.
- Configure **cache settings** and set invalidations when needed.

### 5. **Invalidating CloudFront Cache After Deployment**
If using CloudFront, add the following step to GitHub Actions after the deployment step:

```yaml
      - name: Invalidate CloudFront Cache
        run: |
          aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
```

---

## Troubleshooting & Important Notes
- **Bucket Permissions**: Ensure the S3 bucket policy allows public access if needed.
- **CORS Issues**: If your frontend makes API requests, update the CORS configuration of your S3 bucket.
- **CloudFront Delays**: Changes might not reflect immediately due to caching; use invalidation when required.
- **GitHub Actions Failures**: Check GitHub logs for permission errors and fix secrets if needed.
- **AWS IAM Permissions**: Ensure your IAM user has permissions for `s3:PutObject`, `s3:DeleteObject`, and `cloudfront:CreateInvalidation` if applicable.

---

## Conclusion
This setup ensures an automated deployment workflow where every push to the `main` branch will trigger an update to your S3-hosted web app. If CloudFront is used, caching is also managed automatically.

Let me know if you need further clarifications! 🚀
