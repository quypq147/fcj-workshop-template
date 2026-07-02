---
title: "Prerequisites & Setup"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
# Prerequisites & Project Setup

### 1. Tools Required
Before starting, ensure you have the following installed:
*   **Node.js** (LTS v18 or v20)
*   **AWS CLI v2** configured via `aws configure`
*   **AWS CDK v2** installed globally: `npm install -g aws-cdk`
*   **TypeScript** compiler: `npm install -g typescript`

### 2. Initialize Project
Create a new directory and initialize the CDK app:
```bash
mkdir ecommerce-platform-infra
cd ecommerce-platform-infra
cdk init app --language typescript
```
