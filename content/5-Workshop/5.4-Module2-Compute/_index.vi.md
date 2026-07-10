---
title: "Module 2: Tính toán & API"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Module 2: Compute & API Gateway

**Bước 1: Viết và cấu hình AWS Lambda Services & SQS**
Thiết lập các Lambda Function nghiệp vụ (Product Service, Order Service, Checkout Service). Sử dụng SQS Order Queue làm buffer trước khi Lambda xử lý đơn hàng để chống quá tải.

> 📸 **[HƯỚNG DẪN CHỤP ẢNH THỰC TẾ 4 - DANH SÁCH LÂM BDA FUNCTIONS]**
> *   **Bước 1:** Trên AWS Console, truy cập dịch vụ **AWS Lambda**.
> *   **Bước 2:** Nhấp vào mục **Functions** ở menu bên trái.
> *   **Bước 3:** Sử dụng ô tìm kiếm để lọc các Lambda function có tiền tố của dự án (ví dụ: `MusicStore-` hoặc `AppSingleTable-`).
> *   **Bước 4:** Chụp màn hình danh sách này hiển thị đầy đủ tên các hàm Lambda đã được deploy lên AWS (như ProductService, OrderService, CheckoutService,...).
> *   **Tên file ảnh khuyến nghị lưu:** `/static/images/5-Workshop/lambda_functions_list.png`

**Bước 2: Khởi tạo API Gateway**
Tạo REST API định tuyến request đến các Lambda Function.

![Cây thư mục routes trên API Gateway](/images/5-Workshop/api_gateway_routes.png)
*Hình 5: Cây thư mục và cấu trúc các Route của API Gateway*

---


### Định nghĩa CDK Stack cho Compute & API Gateway
Tạo tệp `lib/api-stack.ts` triển khai API Gateway, cấu hình tích hợp Lambda và phân quyền tương ứng:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { Construct } from 'constructs';

interface ApiStackProps extends cdk.StackProps {
  table: dynamodb.Table;
  userPool: cognito.IUserPool;
}

export class ApiStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: ApiStackProps) {
    super(scope, id, props);

    const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, 'AppApiAuthorizer', {
      cognitoUserPools: [props.userPool],
      authorizerName: 'AppAuthorizer',
    });

    const dataLambda = new lambda.Function(this, 'DataServiceLambda', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromInline(`
        const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
        const { DynamoDBDocumentClient, ScanCommand } = require("@aws-sdk/lib-dynamodb");
        const client = new DynamoDBClient({});
        const docClient = DynamoDBDocumentClient.from(client);

        exports.handler = async (event) => {
          const tableName = process.env.TABLE_NAME;
          try {
            const data = await docClient.send(new ScanCommand({
              TableName: tableName,
              FilterExpression: "begins_with(PK, :pk)",
              ExpressionAttributeValues: { ":pk": "ITEM#" }
            }));
            return {
              statusCode: 200,
              headers: { "Content-Type": "application/json", "Access-Control-Allow-Origin": "*" },
              body: JSON.stringify(data.Items || [])
            };
          } catch (err) {
            return { statusCode: 500, body: JSON.stringify({ error: err.message }) };
          }
        };
      `),
      environment: {
        TABLE_NAME: props.table.tableName,
      },
    });

    props.table.grantReadWriteData(dataLambda);

    const api = new apigateway.RestApi(this, 'AppRestApi', {
      restApiName: 'Application Backend API',
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
      },
    });

    const itemsResource = api.root.addResource('items');
    itemsResource.addMethod('GET', new apigateway.LambdaIntegration(dataLambda));
    itemsResource.addMethod('POST', new apigateway.LambdaIntegration(dataLambda), {
      authorizer: authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO,
    });
  }
}
```

---
