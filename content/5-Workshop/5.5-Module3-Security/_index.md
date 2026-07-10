---
title: "Module 3: Security & Permissions"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# Module 3: Security & Permissions

**Step 1: Identity Management with Amazon Cognito & Authorizers**
Create a Cognito User Pool. Integrate a Cognito Authorizer into API Gateway to protect sensitive routes.

![Cognito Authorizer on API Gateway](/images/5-Workshop/cognito_authorizer.png)
*Figure 6: Cognito Authorizer configuration protecting the API*

**Step 2: Protect the Application with AWS WAF & IAM Least Privilege**
Save the Stripe Secret Key in AWS Secrets Manager. Assign independent IAM Roles to each Lambda. Configure WAF to block bots and apply rate limits.

> 📸 **[PRACTICAL SCREENSHOT GUIDE 7 - AWS WAF WEB ACL RULES]**
> *   **Step 1:** On the AWS Console, open the **WAF & Shield** service.
> *   **Step 2:** Select **Web ACLs** on the left navigation bar and select your region (e.g., `ap-southeast-1` Singapore or `Global CloudFront` depending on your setup).
> *   **Step 3:** Select the Web ACL corresponding to the project (e.g., `MusicStoreWebACL` or `AppWebACL`).
> *   **Step 4:** Switch to the **Rules** tab.
> *   **Step 5:** Take a screenshot of the list of enabled rules (e.g., Core rule set, SQL injection rule set, rate-limiting rule...) currently applied to protect your API Gateway/Amplify.
> *   **Recommended image filename to save:** `/static/images/5-Workshop/waf_web_acls.png`

---

### Define CDK Stack for Security
Create the file `lib/security-stack.ts` to configure the User Pool and set up IAM least privilege permissions:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as iam from 'aws-cdk-lib/aws-iam';
import { Construct } from 'constructs';

export class SecurityStack extends cdk.Stack {
  public readonly userPool: cognito.UserPool;
  public readonly userPoolClient: cognito.UserPoolClient;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.userPool = new cognito.UserPool(this, 'AppUserPool', {
      userPoolName: 'app-user-pool',
      selfSignUpEnabled: true,
      signInAliases: { email: true },
      autoVerify: { email: true },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: true,
      },
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });

    this.userPoolClient = new cognito.UserPoolClient(this, 'AppUserPoolClient', {
      userPool: this.userPool,
      generateSecret: false,
    });

    // Custom role for notification system (Least Privilege)
    const notificationRole = new iam.Role(this, 'NotificationPublishRole', {
      assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
      description: 'Role to publish notifications with minimum permissions',
    });

    notificationRole.addToPolicy(new iam.PolicyStatement({
      actions: ['sns:Publish'],
      resources: ['*'],
      effect: iam.Effect.ALLOW,
    }));
  }
}
```
