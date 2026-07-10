---
title: "Deployment & Testing"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
# Deployment & Testing (End-to-End Test)

**Step 1: Deploy the Backend Using CDK**
To push the entire infrastructure to AWS, run the following commands in the root directory of your CDK project:

```bash
# Bootstrap auxiliary resources on the specified region
cdk bootstrap

# Verify the difference between local code and cloud
cdk diff

# Deploy all stacks to AWS Cloud
cdk deploy --all
```

![CDK Backend Deployment Results](/images/5-Workshop/cdk_deploy_outputs.png)
*Figure 8: Terminal log upon successful execution of cdk deploy*

**Step 2: Deploy the Frontend with AWS Amplify**
Connect the GitHub repository containing the Next.js source code to AWS Amplify, configure the required environment variables from the Outputs in Step 1, and run the Build & Deploy process (supports the `WEB_COMPUTE` platform for Next.js SSR).

![Deployment Process on AWS Amplify](/images/5-Workshop/amplify_build_deploy.png)
*Figure 9: Successful Next.js frontend deployment status on AWS Amplify*

**Step 3: Perform End-to-End (E2E) Testing**
Access the domain name assigned by Amplify for the website. Perform a product purchase, complete the checkout process, and monitor the system processing flow logs.

![Successful Payment & CloudWatch Logs](/images/5-Workshop/e2e_test_success.png)
*Figure 10: Successful payment confirmation and event tracking in CloudWatch logs*
