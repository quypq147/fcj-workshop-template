---
title: "Module 4: Serverless Backend"
date: 2026-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Module 4: Serverless Backend & Application Programming (AWS SDK v3)

In this module, you will implement the asynchronous data ingestion and event processing logic using **AWS SDK v3** in TypeScript. This decoupled architecture uses API Gateway, Lambda, DynamoDB, Amazon SQS, and Amazon EventBridge.

---

## Part 1: Overview & Prerequisites

### 1. Architectural Flow
To support high-throughput, decoupled, and reliable request processing, the platform implements an **event-driven architecture**:
1. The **Ingestion Lambda** receives a request from API Gateway.
2. It performs a **DynamoDB Conditional Write** to store the initial state as "PENDING" and registers the client-provided `Idempotency-Key` to prevent duplicate processing.
3. It integrates with an external third-party service (simulated).
4. Upon success, it publishes a `data.received` event to an **Amazon EventBridge Custom Bus**.
5. EventBridge routes this event to an **SQS Processing Queue**.
6. The **Worker Lambda (SQS Consumer)** polls the queue, checks for duplicates, updates the DynamoDB record to "COMPLETED", and handles failures.
7. Any persistently failing messages are sent to the **Dead-Letter Queue (DLQ)**.

```mermaid
graph LR
    Client -->|1. POST /ingest| APIGW[API Gateway]
    APIGW -->|2. Invoke| IngestionLambda[Ingestion Lambda]
    IngestionLambda -->|3. Conditional Write| DDB[(DynamoDB Single-Table)]
    IngestionLambda -->|4. Publish Event| EB[EventBridge Custom Bus]
    EB -->|5. Route Event| SQS[SQS Processing Queue]
    SQS -->|6. Poll Batch| WorkerLambda[Worker Lambda]
    WorkerLambda -->|7. Update Status| DDB
    WorkerLambda -.->|Failed Messages| DLQ[SQS Dead-Letter Queue]
```

### 2. NPM Dependencies
AWS SDK v3 is modular, meaning you only install the client libraries you actually use. This reduces the Lambda package size and improves cold-start times.

Run the following command in your Lambda service directory:
```bash
npm install @aws-sdk/client-dynamodb @aws-sdk/client-sqs @aws-sdk/client-eventbridge @aws-sdk/lib-dynamodb
npm install --save-dev @types/aws-lambda typescript
```

---

## Part 2: Step-by-Step SDK Implementation

### 1. Ingestion Service Lambda (Event Publisher)
Create the ingestion function to validate the request, save the initial state with conditional constraints to avoid duplicates, and publish the event.

Create `services/ingestion/index.ts`:
```typescript
import { APIGatewayProxyEvent, APIGatewayProxyResult } from "aws-lambda";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";
import { EventBridgeClient, PutEventsCommand } from "@aws-sdk/client-eventbridge";

// Initialize SDK clients (Clients are reused across Lambda invocations)
const ddbClient = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(ddbClient);
const eventBridgeClient = new EventBridgeClient({});

const TABLE_NAME = process.env.TABLE_NAME || "";
const EVENT_BUS_NAME = process.env.EVENT_BUS_NAME || "";

export const handler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  console.log("Received event:", JSON.stringify(event));

  try {
    if (!event.body) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: "Missing request body" }),
      };
    }

    const { id, userId, dataPayload, idempotencyKey } = JSON.parse(event.body);

    if (!id || !userId || !dataPayload || !idempotencyKey) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: "Missing required fields: id, userId, dataPayload, idempotencyKey" }),
      };
    }

    // 1. Store initial status with DynamoDB Conditional Write
    // Use the idempotencyKey to prevent duplicate creations
    const pk = `INGESTION#${id}`;
    const sk = "METADATA";

    console.log(`Attempting conditional write for PK: ${pk}`);
    
    try {
      await docClient.send(
        new PutCommand({
          TableName: TABLE_NAME,
          Item: {
            PK: pk,
            SK: sk,
            userId,
            dataPayload,
            status: "PENDING",
            idempotencyKey,
            createdAt: new Date().toISOString(),
            updatedAt: new Date().toISOString(),
          },
          // Ensure we don't overwrite if PK & SK already exist (guarantees idempotency)
          ConditionExpression: "attribute_not_exists(PK)",
        })
      );
    } catch (dbError: any) {
      if (dbError.name === "ConditionalCheckFailedException") {
        console.warn(`Duplicate submission detected for ID: ${id}`);
        return {
          statusCode: 409,
          body: JSON.stringify({ error: "Request already exists or is being processed" }),
        };
      }
      throw dbError;
    }

    // 2. Simulate 3rd Party Integration
    console.log(`Processing simulated integration for payload ID: ${id}`);
    const isIntegrationSuccessful = await simulateThirdPartyCall(id);
    if (!isIntegrationSuccessful) {
      throw new Error("External service integration failed");
    }

    // 3. Publish data.received event to EventBridge Custom Bus
    console.log(`Publishing data.received event to custom bus: ${EVENT_BUS_NAME}`);
    const eventPayload = {
      id,
      userId,
      dataPayload,
      status: "INGESTED",
      idempotencyKey,
    };

    await eventBridgeClient.send(
      new PutEventsCommand({
        Entries: [
          {
            Source: "app.ingestion",
            DetailType: "data.received",
            Detail: JSON.stringify(eventPayload),
            EventBusName: EVENT_BUS_NAME,
          },
        ],
      })
    );

    return {
      statusCode: 201,
      headers: { "Access-Control-Allow-Origin": "*" },
      body: JSON.stringify({
        message: "Request ingested successfully",
        id,
        status: "PENDING",
      }),
    };
  } catch (error: any) {
    console.error("Error in ingestion handler:", error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message || "Internal Server Error" }),
    };
  }
};

// Simulated external API call
async function simulateThirdPartyCall(id: string): Promise<boolean> {
  // Simulate network latency
  await new Promise((resolve) => setTimeout(resolve, 500));
  // Fail integration if ID contains "fail" for testing
  if (id.includes("fail")) {
    return false;
  }
  return true;
}
```

### 2. Worker Lambda Consumer (SQS Consumer)
Create the asynchronous processing worker that reads SQS message batches, updates DynamoDB, and handles idempotency check at the consumer level.

Create `services/worker/index.ts`:
```typescript
import { SQSEvent, SQSBatchResponse, SQSRecord } from "aws-lambda";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, UpdateCommand, PutCommand } from "@aws-sdk/lib-dynamodb";

const ddbClient = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(ddbClient);
const TABLE_NAME = process.env.TABLE_NAME || "";

export const handler = async (event: SQSEvent): Promise<SQSBatchResponse> => {
  console.log(`Processing batch of ${event.Records.length} records`);
  
  const batchItemFailures: { itemIdentifier: string }[] = [];

  for (const record of event.Records) {
    try {
      await processMessage(record);
    } catch (err) {
      console.error(`Failed to process message ${record.messageId}:`, err);
      // Track failed message IDs to return to SQS for retry
      batchItemFailures.push({ itemIdentifier: record.messageId });
    }
  }

  // Returning batchItemFailures tells SQS only to retry the failed messages,
  // preventing reprocessing of successful messages in this batch.
  return { batchItemFailures };
};

async function processMessage(record: SQSRecord): Promise<void> {
  const messageBody = JSON.parse(record.body);
  
  // Extract custom event data forwarded from EventBridge
  const details = messageBody.detail;
  if (!details || !details.id) {
    console.warn(`Invalid message schema, skipping. Message ID: ${record.messageId}`);
    return;
  }

  const { id, userId, dataPayload } = details;

  // 1. Idempotent Consumer Pattern: Save message process tracking key
  // Avoid duplicate processing of the same SQS message
  const idempotencyPk = `IDEMPOTENCY#MSG#${record.messageId}`;
  const idempotencySk = `INGESTION#${id}`;

  try {
    await docClient.send(
      new PutCommand({
        TableName: TABLE_NAME,
        Item: {
          PK: idempotencyPk,
          SK: idempotencySk,
          processedAt: new Date().toISOString(),
          ttl: Math.floor(Date.now() / 1000) + 86400, // 24-hour retention
        },
        ConditionExpression: "attribute_not_exists(PK)",
      })
    );
  } catch (dbError: any) {
    if (dbError.name === "ConditionalCheckFailedException") {
      console.warn(`SQS Message already processed: ${record.messageId}. Skipping.`);
      return; // Skip execution safely
    }
    throw dbError;
  }

  // 2. Simulate Failure Scenario for DLQ Testing
  // If dataPayload contains a failProcessing flag, force fail processing to trigger retry -> DLQ
  if (dataPayload && dataPayload.failProcessing === true) {
    throw new Error(`[SIMULATED_FAILURE] Processing failed for request ${id} due to failProcessing flag`);
  }

  // 3. Update Status to "COMPLETED" in DynamoDB
  console.log(`Updating status to COMPLETED for ingestion ID: ${id}`);
  await docClient.send(
    new UpdateCommand({
      TableName: TABLE_NAME,
      Key: {
        PK: `INGESTION#${id}`,
        SK: "METADATA",
      },
      UpdateExpression: "SET status = :status, updatedAt = :updatedAt",
      ExpressionAttributeValues: {
        ":status": "COMPLETED",
        ":updatedAt": new Date().toISOString(),
      },
    })
  );

  console.log(`Successfully completed processing for ID: ${id}`);
}
```

---

## Part 3: IAM Permissions & SQS Configurations

### 1. IAM Configurations (Principle of Least Privilege)
To ensure the Lambda functions only have access to what they need, declare explicit IAM roles instead of administrative permissions.

Here is the configuration logic represented in **AWS CDK**:

```typescript
import * as iam from "aws-cdk-lib/aws-iam";
import * as sqs from "aws-cdk-lib/aws-sqs";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import * as events from "aws-cdk-lib/aws-events";

// A. Ingestion Lambda Role Permissions
const ingestionLambda = new lambda.Function(this, "IngestionLambda", { /* ... */ });

// Grant Ingestion Lambda permissions to put items in DynamoDB
ingestionLambda.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: [
    "dynamodb:PutItem",
  ],
  resources: [props.table.tableArn],
}));

// Grant Ingestion Lambda permissions to publish events to Custom EventBus
ingestionLambda.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: [
    "events:PutEvents",
  ],
  resources: [props.customEventBus.eventBusArn],
}));


// B. Worker Lambda Role Permissions
const workerLambda = new lambda.Function(this, "WorkerLambda", { /* ... */ });

// Grant Worker Lambda permissions to query, update items in DynamoDB
workerLambda.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: [
    "dynamodb:PutItem",
    "dynamodb:UpdateItem",
  ],
  resources: [props.table.tableArn],
}));

// Grant SQS Consumer Lambda permissions to receive and delete messages
workerLambda.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: [
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage",
    "sqs:GetQueueAttributes"
  ],
  resources: [props.processingQueue.queueArn],
}));
```

### 2. SQS Timeout and Visibility Settings
When integrating SQS with Lambda, the **Visibility Timeout** of the SQS Queue is a crucial reliability parameter. 

```typescript
const processingDlq = new sqs.Queue(this, "ProcessingDLQ", {
  queueName: "processing-dlq",
  retentionPeriod: cdk.Duration.days(14), // Retain for debugging
});

const processingQueue = new sqs.Queue(this, "ProcessingQueue", {
  queueName: "processing-queue",
  // Crucial: Must be greater than or equal to Lambda's Timeout
  visibilityTimeout: cdk.Duration.seconds(180), // 3 minutes
  deadLetterQueue: {
    queue: processingDlq,
    maxReceiveCount: 3, // Move to DLQ after 3 failures
  },
});
```

#### Why `visibilityTimeout` must be greater than Lambda's timeout:
- **The Concept**: When Lambda polls SQS, it pulls messages and locks them for the duration of the `visibilityTimeout`.
- **The Issue**: If the Lambda function's timeout is set to 30 seconds, but the SQS Visibility Timeout is set to 15 seconds, and a message takes 20 seconds to process:
  1. The visibility lock expires at 15 seconds.
  2. SQS makes the message visible again to other pollers.
  3. Another Lambda instance picks up the exact same message while the first instance is still running.
  4. This causes duplicate executions and potential race conditions.
- **The Best Practice**: Set the SQS `visibilityTimeout` to **at least 6 times** the Lambda function timeout plus any batch window buffer. 

---

## Part 4: Testing & Observability

### 1. Test Payloads

#### Test A: Triggering Ingestion Service (API Gateway Request)
Use this JSON payload to test the `/ingest` POST endpoint.

```json
{
  "id": "REQ-2026-999-TEST",
  "userId": "usr_998877",
  "idempotencyKey": "uuid-550e8400-e29b-41d4-a716-446655440000",
  "dataPayload": {
    "key": "val",
    "someNumeric": 100.5
  }
}
```

*Expected Output (First request)*:
`201 Created` with body `{"message":"Request ingested successfully","id":"REQ-2026-999-TEST","status":"PENDING"}`

*Expected Output (Second request with same body)*:
`409 Conflict` with body `{"error":"Request already exists or is being processed"}` (Idempotence works!).

#### Test B: Simulating SQS Dead-Letter Queue (DLQ) Movement
To test DLQ ingestion, send a request triggering the processing error. Set the payload `failProcessing` parameter to `true`.

```json
{
  "id": "REQ-2026-FAIL-DLQ",
  "userId": "usr_fail_test",
  "idempotencyKey": "uuid-fail-dlq-test-001",
  "dataPayload": {
    "failProcessing": true
  }
}
```

*Expected Flow*:
1. The Ingestion Lambda executes, registers the request, and publishes the event.
2. The Worker Lambda picks up the message from SQS.
3. The Worker throws `[SIMULATED_FAILURE] Processing failed for request...`.
4. The message returns to the queue. This repeats 3 times (`maxReceiveCount`).
5. After the 3rd attempt, the message is automatically moved to `processing-dlq`.

---

### 2. Observability & Tracing

#### A. Structured Logging with CloudWatch
Using `console.log(JSON.stringify(event))` creates **Structured JSON Logs** in CloudWatch. This allows you to run **CloudWatch Logs Insights** queries like:

```sql
fields @timestamp, @message, id, error
| filter @message like /Error/ or @message like /ConditionalCheckFailedException/
| sort @timestamp desc
| limit 20
```

#### B. Distributed Tracing with AWS X-Ray
To trace a request end-to-end (API Gateway -> SQS -> Lambda -> EventBridge -> DynamoDB):

1. **Enable Active Tracing** in CDK for Lambda functions:
   ```typescript
   const ingestionLambda = new lambda.Function(this, "IngestionLambda", {
     // ...
     tracing: lambda.Tracing.ACTIVE
   });
   ```
2. **Enable Tracing on API Gateway Stage**:
   ```typescript
   const api = new apigateway.RestApi(this, "AppRestApi", {
     deployOptions: {
       tracingEnabled: true
     }
   });
   ```
3. **Analyze the Trace Map**:
   Navigate to AWS console -> **CloudWatch** -> **X-Ray traces** -> **Service map**. You will see the visual flow showing latency metrics, error rates, and communication links across each hops.
