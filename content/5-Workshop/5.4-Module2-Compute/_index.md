---
title: "Module 2: Compute & API"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Module 2: Compute & API Gateway

Wire a backend Lambda function and expose it through API Gateway. Since we are using a fully serverless architecture, the Lambda function runs in the default AWS-managed network and communicates directly with DynamoDB.

Create `lib/api-stack.ts`:
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
