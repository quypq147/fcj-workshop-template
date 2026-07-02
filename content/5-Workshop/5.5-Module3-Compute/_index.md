---
title: "Module 3: Compute & API"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# Module 3: Compute & API Gateway

Wire backend Lambda function in private subnet and expose it through API Gateway.

Create `lib/api-stack.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { Construct } from 'constructs';

interface ApiStackProps extends cdk.StackProps {
  vpc: ec2.IVpc;
  table: dynamodb.Table;
  userPool: cognito.IUserPool;
}

export class ApiStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: ApiStackProps) {
    super(scope, id, props);

    const authorizer = new apigateway.CognitoUserPoolsAuthorizer(this, 'ECommerceApiAuthorizer', {
      cognitoUserPools: [props.userPool],
      authorizerName: 'ECommerceAuthorizer',
    });

    const productLambda = new lambda.Function(this, 'ProductServiceLambda', {
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
              ExpressionAttributeValues: { ":pk": "PRODUCT#" }
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
      vpc: props.vpc,
      vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
      environment: {
        TABLE_NAME: props.table.tableName,
      },
    });

    props.table.grantReadWriteData(productLambda);

    const api = new apigateway.RestApi(this, 'ECommerceRestApi', {
      restApiName: 'E-Commerce Backend API',
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
      },
    });

    const productsResource = api.root.addResource('products');
    productsResource.addMethod('GET', new apigateway.LambdaIntegration(productLambda));
    productsResource.addMethod('POST', new apigateway.LambdaIntegration(productLambda), {
      authorizer: authorizer,
      authorizationType: apigateway.AuthorizationType.COGNITO,
    });
  }
}
```
