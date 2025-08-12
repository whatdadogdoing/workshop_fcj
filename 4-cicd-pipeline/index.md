---
layout: default
title: "Module 4: CI/CD Pipeline Implementation"
---

# Module 4: CI/CD Pipeline Implementation

## Overview
Build a complete enterprise-grade CI/CD pipeline using GitHub Actions with security scanning, testing, and automated deployment.

## Learning Objectives
- Create GitHub Actions workflows
- Implement security scanning with Trivy
- Set up automated testing
- Configure deployment to AWS ECS
- Add monitoring and notifications

## Pipeline Architecture

![CI/CD Pipeline Flow](../images/pipeline-flow.png)

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
│   Code      │───▶│   Security   │───▶│   Testing   │───▶│   Build &   │
│   Push      │    │   Scanning   │    │   & QA      │    │   Deploy    │
└─────────────┘    └──────────────┘    └─────────────┘    └─────────────┘
```

## Implementation Steps

### 4.1 Create Main Pipeline

Create `.github/workflows/main.yml`:

```yaml
name: Enterprise CI/CD Pipeline

on:
  push:
    branches: [main, develop, staging]
  pull_request:
    branches: [main, develop]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: enterprise-app

jobs:
  security-scan:
    name: Security & Vulnerability Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
        continue-on-error: true

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    strategy:
      matrix:
        component: [backend, frontend]
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        if: matrix.component == 'backend'
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Setup Node.js
        if: matrix.component == 'frontend'
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Run Backend Tests
        if: matrix.component == 'backend'
        run: |
          cd backend
          pip install -r requirements.txt
          pytest tests/ -v --cov=. --cov-report=xml
        continue-on-error: true
      
      - name: Run Frontend Tests
        if: matrix.component == 'frontend'
        run: |
          cd frontend
          npm ci
          npm test -- --coverage --watchAll=false
        continue-on-error: true

  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    needs: [security-scan, test]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build and push Docker images
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build and push backend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY-backend:$IMAGE_TAG ./backend
          docker push $ECR_REGISTRY/$ECR_REPOSITORY-backend:$IMAGE_TAG
          
          # Build and push frontend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY-frontend:$IMAGE_TAG ./frontend
          docker push $ECR_REGISTRY/$ECR_REPOSITORY-frontend:$IMAGE_TAG
      
      - name: Deploy to ECS
        run: |
          # Update ECS service with new image
          aws ecs update-service \
            --cluster enterprise-cluster \
            --service enterprise-service \
            --force-new-deployment
```

### 4.2 GitHub Secrets Setup

Add these secrets to your repository:

| Secret | Description | Example |
|--------|-------------|---------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Key | `wJalrXUtnFEMI/K7MDENG/...` |
| `SONAR_TOKEN` | SonarCloud Token | `sqp_1234567890abcdef...` |
| `SLACK_WEBHOOK_URL` | Slack Webhook | `https://hooks.slack.com/...` |

### 4.3 Testing the Pipeline

```bash
# Make a change and push
echo "# Pipeline Test" >> README.md
git add README.md
git commit -m "test: trigger CI/CD pipeline"
git push origin main
```

## Validation Checklist

- [ ] Pipeline triggers on push/PR
- [ ] Security scanning completes
- [ ] Tests run successfully
- [ ] Docker images build and push
- [ ] ECS deployment succeeds
- [ ] Notifications work
- [ ] Rollback mechanism tested

## Next Steps

Pipeline is ready! Proceed to:
**[Module 5: AWS Infrastructure →](../5-aws-infrastructure/)**

---

**Pipeline failing?** Check the [troubleshooting guide](../troubleshooting#pipeline-issues) for common solutions.