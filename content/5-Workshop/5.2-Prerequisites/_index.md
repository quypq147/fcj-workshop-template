---
title: "Prerequisites & Setup"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
# Prerequisites & Project Setup

Before starting, ensure your environment is set up:
*   Install Node.js, AWS CLI, and AWS CDK.
*   Configure credentials with AWS (using `aws configure`).

![AWS Account Credentials](/images/5-Workshop/aws_configure_sts.png)
*Figure 2: AWS credentials configuration and identity check results (get-caller-identity)*

### Initialize CDK Project
Create a new directory and initialize the CDK app with TypeScript:
```bash
mkdir ecommerce-platform-infra
cd ecommerce-platform-infra
cdk init app --language typescript
```
