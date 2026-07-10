---
title: "Module 2: Compute & API"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Module 2: Compute & API Gateway

**Step 1: Write and Configure AWS Lambda Services & SQS**
Set up the business Lambda Functions (Product Service, Order Service, Checkout Service). Use an SQS Order Queue as a buffer before Lambda processes orders to prevent overload.

> 📸 **[PRACTICAL SCREENSHOT GUIDE 4 - LIST OF LAMBDA FUNCTIONS]**
> *   **Step 1:** On the AWS Console, access the **AWS Lambda** service.
> *   **Step 2:** Click on **Functions** in the left menu.
> *   **Step 3:** Use the search box to filter Lambda functions with the project prefix (e.g., `MusicStore-` or `AppSingleTable-`).
> *   **Step 4:** Take a screenshot of this list displaying all Lambda functions deployed to AWS (such as ProductService, OrderService, CheckoutService,...).
> *   **Recommended image filename to save:** `/static/images/5-Workshop/lambda_functions_list.png`

**Step 2: Initialize API Gateway**
Create a REST API to route requests to the Lambda Functions.

![API Gateway Routes Tree](/images/5-Workshop/api_gateway_routes.png)
*Figure 5: Tree structure and configuration of API Gateway routes*

---

### Define CDK Stack for Compute & API Gateway
Create the file `lib/api-stack.ts` to implement API Gateway, configure Lambda integrations, and set up corresponding permissions:

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
