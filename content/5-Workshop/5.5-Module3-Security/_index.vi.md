---
title: "Module 3: Bảo mật & Phân quyền"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# Module 3: Security & Permissions

**Bước 1: Quản lý danh tính với Amazon Cognito & Authorizer**
Tạo Cognito User Pool. Tích hợp Cognito Authorizer vào API Gateway để bảo vệ các route nhạy cảm.

![Cognito Authorizer trên API Gateway](/images/5-Workshop/cognito_authorizer.png)
*Hình 6: Cấu hình Cognito Authorizer bảo vệ API*

**Bước 2: Bảo vệ ứng dụng với AWS WAF & IAM Least Privilege**
Lưu Stripe Secret Key vào AWS Secrets Manager. Phân quyền IAM Role độc lập cho từng Lambda. Cấu hình WAF để chặn bot và rate limit.

> 📸 **[HƯỚNG DẪN CHỤP ẢNH THỰC TẾ 7 - AWS WAF WEB ACL RULES]**
> *   **Bước 1:** Trên AWS Console, mở dịch vụ **WAF & Shield**.
> *   **Bước 2:** Chọn **Web ACLs** ở thanh điều hướng bên trái và chọn vùng của bạn (`ap-southeast-1` Singapore hoặc `Global CloudFront` tùy theo cấu hình của bạn).
> *   **Bước 3:** Chọn Web ACL tương ứng với dự án (ví dụ: `MusicStoreWebACL` hoặc `AppWebACL`).
> *   **Bước 4:** Chuyển sang tab **Rules** (Quy tắc).
> *   **Bước 5:** Thực hiện chụp màn hình danh sách các quy tắc (ví dụ: Core rule set, SQL injection rule set, rate-limiting rule...) đang được kích hoạt và áp dụng để bảo vệ API Gateway/Amplify.
> *   **Tên file ảnh khuyến nghị lưu:** `/static/images/5-Workshop/waf_web_acls.png`

---


### Định nghĩa CDK Stack cho Security
Tạo tệp `lib/security-stack.ts` để cấu hình User Pool và phân quyền tối thiểu IAM:

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

    // Custom role cho hệ thống thông báo (Quyền tối thiểu)
    const notificationRole = new iam.Role(this, 'NotificationPublishRole', {
      assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
      description: 'Role gửi thông báo với các quyền tối giản',
    });

    notificationRole.addToPolicy(new iam.PolicyStatement({
      actions: ['sns:Publish'],
      resources: ['*'],
      effect: iam.Effect.ALLOW,
    }));
  }
}
```

