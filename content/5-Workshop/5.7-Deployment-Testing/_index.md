---
title: "Deployment & Testing"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
# Deployment & Testing

### 1. Deployment Steps
Execute the following commands in the infrastructure project root:
```bash
# Initialize CDK bootstrap resources
cdk bootstrap

# Compile and compare diffs
cdk diff

# Deploy all stacks to AWS
cdk deploy --all
```

### 2. Testing APIs
Once deploy completes, API Gateway outputs the endpoint URL. Use `curl` to test:
```bash
# Public GET list of products
curl -X GET https://<API-ID>.execute-api.us-east-1.amazonaws.com/prod/products
```
