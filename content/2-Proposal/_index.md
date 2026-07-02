---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SYSTEM ARCHITECTURE PROPOSAL
## PROJECT: AI E-COMMERCE PLATFORM — AWS SERVERLESS ARCHITECTURE

---

### 1. INTRODUCTION & PROBLEM STATEMENT

#### 1.1. Cloud-native and Decoupled Architecture Trends
Entering 2026, the global e-commerce industry is witnessing a massive transition from traditional monolithic architectures to **Cloud-native** and **decoupled/event-driven architectures**. Fierce competition demands that online retail platforms adapt rapidly to market shifts, handle massive traffic spikes during peak sales seasons without downtime, and ensure high operational agility.

A decoupled architecture separates the user interface (Frontend) from the core business logic (Backend). By integrating event-driven design patterns, the system can asynchronously process heavy computational workloads, avoiding blocking synchronous interactions, thereby achieving maximum fault tolerance and system resilience.

#### 1.2. Pain Points of Traditional Architectures
When operating conventional e-commerce systems—often deployed on self-managed virtual machines or monolithic relational databases—organizations frequently encounter key challenges:
- **Connection Limits under Spike Traffic:** During flash sales or marketing campaigns, traffic surges can exhaust database and web server connection limits, causing bottlenecks and rendering the site unresponsive.
- **High Idle Costs and Resource Waste:** Infrastructure must be provisioned to handle peak capacities, leading to significant resource idling and high costs during off-peak hours.
- **Double Charging Risks:** Flaky network connections or rapid multiple clicks on payment buttons can trigger duplicate payment transactions because of a lack of idempotency mechanisms, damaging brand reputation.
- **Security Vulnerabilities:** Managing infrastructure manually increases vulnerability to DDoS attacks, web application exploits (SQL Injection, XSS), and data breaches due to the absence of edge filtering and fine-grained access control.
- **Complex Order State Management and Syncing:** In tightly-coupled systems, a failure in the notification or catalog updating services can directly block the user checkout flow, resulting in lost sales and decreased conversion rates.

---

### 2. PROJECT OBJECTIVES

To address these critical challenges, this project proposes the design and implementation of an **AI E-Commerce Platform** leveraging AWS Serverless infrastructure with the following goals:
- **Autoscaling Capabilities:** Ensure the system scales seamlessly in real-time to support sudden traffic bursts without manual operational intervention.
- **Pay-as-you-go Pricing:** Utilize a cost-optimized model based on actual resource consumption, eliminating idle-time infrastructure expenses.
- **Robust Core Business Workflows:**
  - Fast catalog listing and product image delivery via Amazon S3 integrated with a CDN.
  - Secure identity verification, sign-up, sign-in, and authorization using Amazon Cognito (issuing JWT tokens).
  - Decoupled, asynchronous order management to offload processing tasks from frontend clients.
  - Secure payment processing via Stripe Sandbox, preventing duplicate payments via `Idempotency-Key` and securing webhook integration with signature validation.
  - AI-driven shopping assistance with Amazon Lex chatbot and asynchronous multi-channel notification handlers (Email/Push/In-app).

---

### 3. PROPOSED SYSTEM ARCHITECTURE

#### 3.1. Rationale for AWS Serverless Architecture
Instead of using containerized microservices (such as ECS or EKS) which require heavy container orchestration and OS maintenance, this project adopts an **Advanced AWS Serverless Architecture (v3)**:
- **Reduced Blast Radius:** Each serverless function (AWS Lambda) is isolated and focused on a single business domain. A failure in one service (e.g., notification delivery) is isolated and does not disrupt core checkout or catalog lookup flows.
- **Enhanced Developer Experience (DX):** Developers focus entirely on core business logic rather than network configuration, OS patching, or provisioning load balancers. Infrastructure as Code (IaC) via AWS CDK enables unified, reproducible deployments and automated CI/CD.
- **High Availability & Durability:** Automated multi-AZ replication is built natively into services like API Gateway, Lambda, SQS, EventBridge, and DynamoDB.

#### 3.2. Official System Architecture Diagram (v3)

![AWS Serverless E-commerce Architecture v3](/images/2-Proposal/architecture.png)

> **Implementation Note:** AWS WAF is logically bound at the edge to protect API endpoints and content delivery, linking with AWS Amplify's CloudFront distribution and Amazon API Gateway REST endpoints.

#### 3.3. Detailed Data Flow
The architecture executes 7 primary, optimized data flows:
1. **Flow 1 - Website Access:** User resolves domain via Route 53 -> AWS WAF evaluates traffic via managed rule groups and rate limits -> AWS Amplify serves Next.js SSR/Static assets via CDN.
2. **Flow 2 - Authentication:** Users sign up/in using Cognito Hosted UI or SDK -> Cognito issues a secure JWT -> Frontend sends JWT in `Authorization: Bearer <token>` headers -> API Gateway Cognito Authorizer validates JWT before routing to Lambda.
3. **Flow 3 - Catalog Query:** Frontend requests `GET /products` -> API Gateway -> Product Service (Lambda) queries metadata from DynamoDB and generates secure S3 pre-signed URLs for product images.
4. **Flow 4 - Async Order Flow:** User requests order placement -> API Gateway -> Order Service (Lambda) records initial state in DynamoDB -> Publishes message to **SQS Order Queue** -> Consumer Lambda processes and deletes message on success -> Message goes to **Order DLQ** on persistent failure.
5. **Flow 5 - Checkout & Payment:** Frontend initiates `POST /checkout/payment-intent` -> Checkout Service (Lambda) evaluates cart and talks to Stripe API -> Creates Stripe Payment Intent using a unique `Idempotency-Key` -> Stores reference in DynamoDB -> Stripe Webhook returns payment status (verified via webhook signature) -> Checkout Service publishes `order.placed` event to Amazon EventBridge.
6. **Flow 6 - Event-Driven Updates & Notifications:** EventBridge Custom Bus routes `order.placed` events to targets:
   - Rule 1: Order Service updates status to "Paid" in DynamoDB.
   - Rule 2: Sends message to **Notification Queue** -> Notification Lambda delivers message -> Failed attempts go to **Notification DLQ**.
7. **Flow 7 - AI Chatbot:** User queries Chatbot UI -> API Gateway -> Chatbot Backend (Lambda) -> Invokes Amazon Lex Runtime to resolve intents and replies to frontend, querying internal APIs securely via restricted IAM roles when database lookup is needed.

#### 3.4. Security Control Points
- **Cognito Authorizer:** Protects private endpoints at the API Gateway level.
- **Encryption at Rest & Transit:** Enforced using AWS KMS for DynamoDB, S3, and Lambda environment variables.
- **Threat Detection:** Amazon GuardDuty monitors API calls, CloudTrail logs, and network behavior to detect active threats.
- **Least Privilege:** Each Lambda function has a dedicated IAM Role with minimal necessary permissions.
- **Secrets Management:** Sensitive keys (Stripe API Keys) are stored in AWS Secrets Manager and never committed to version control.

---

### 4. TEAM DIVISION & WORKLOAD

Workload is distributed equitably among the 5 team members to ensure parallel progress:

| Member | Role | Key Responsibilities | Deliverables |
|---|---|---|---|
| **Member 1** | **Frontend & Identity Engineer** | - Build Next.js UI hosted on AWS Amplify.<br>- Integrate Cognito Hosted UI and SDK for auth workflows.<br>- Create Cart, Checkout, and Chatbot UI components.<br>- Implement client-side route protection. | - Frontend codebase.<br>- Complete sign-in/up UI flow.<br>- Checkout & chatbot interface.<br>- Amplify deployment pipelines. |
| **Member 2** | **API & Compute Engineer** | - Configure REST API Gateway and Cognito Authorizer.<br>- Develop Product Service Lambda (CRUD).<br>- Develop Order Service REST API.<br>- Implement incoming request body validation. | - OpenAPI contract specs.<br>- Product & Order Lambda codebase.<br>- API Gateway configurations.<br>- Unit testing reports. |
| **Member 3** | **Data & Messaging Engineer** | - Design DynamoDB Single-Table Schema & GSIs.<br>- Structure S3 product asset storage.<br>- Configure SQS Order Queue + DLQ.<br>- Wire EventBridge Custom Bus & Routing rules.<br>- Setup automated DynamoDB backup policies via AWS Backup. | - DynamoDB single-table design docs.<br>- SQS, DLQ & EventBridge configuration script.<br>- Data storage layout.<br>- SQS recovery/restore logs & AWS Backup configuration. |
| **Member 4** | **Payment, AI & Notification Engineer** | - Develop Checkout Service Lambda and Stripe webhook integration.<br>- Train Amazon Lex Intents, Utterances, and Slots.<br>- Develop Notification Service and Lambda consumers. | - Checkout Lambda & Webhook handler codebase.<br>- Configured Amazon Lex chatbot model.<br>- Notification consumer handler code.<br>- Payment & notification integration tests. |
| **Member 5** | **DevSecOps & Observability (Team Lead / PM)** | - Manage IaC templates via AWS CDK.<br>- Enforce least privilege IAM profiles.<br>- Configure edge security: AWS WAF, Route 53, and Secrets Manager.<br>- Build CI/CD pipelines (GitHub Actions).<br>- Configure CloudWatch dashboards, alarms, X-Ray tracing, and GuardDuty.<br>- Manage project timelines, sync sessions, and final demos. | - CDK IaC codebase.<br>- Automated CI/CD GitHub workflows.<br>- CloudWatch dashboards & alert metrics.<br>- Security reports (GuardDuty, WAF).<br>- Project management assets (Runbook, Release Plan). |

---

### 5. BUDGET ESTIMATION (Monthly Development/Lab Scale)

The following estimate details cost projection for 1 month of lab development to prove Serverless economy:

| AWS Service | Estimated Usage / Month | Billing Formula | Estimated Cost / Month |
|---|---|---|---|
| **AWS Amplify** | - 100 build minutes/month<br>- 1 GB storage, 5 GB bandwidth | - Build: $0.01/min<br>- Host: $0.02/GB storage, $0.15/GB traffic | $1.00 - $1.75 |
| **Amazon Cognito** | - 50 Monthly Active Users (MAU) | - Free Tier (up to 50,000 MAU) | $0.00 |
| **Amazon API Gateway** | - 20,000 requests/month | - $3.50 per million REST API requests | $0.07 |
| **AWS Lambda** | - 30,000 executions (500ms avg, 512MB RAM) | - Free Tier (1M requests & 400,000 GB-s) | $0.00 |
| **Amazon DynamoDB** | - 2 GB data storage<br>- 100,000 read/write requests (On-demand) | - Storage: $0.25/GB<br>- Requests: $1.25/M writes, $0.25/M reads | $0.50 |
| **Amazon S3** | - 5 GB product media storage<br>- 5,000 request ops | - Storage: $0.023/GB<br>- Requests: $0.005/1,000 PUT, $0.0004/1,000 GET | $0.15 |
| **Amazon SQS & DLQ** | - 10,000 messages/month | - Free Tier (first 1M messages) | $0.00 |
| **Amazon EventBridge** | - 5,000 events/month | - Free Tier (first 1M custom events) | $0.00 |
| **AWS Secrets Manager** | - 1 active secret (Stripe key) | - $0.40/secret/month<br>- $0.05/10,000 API calls | $0.45 |
| **AWS WAF** | - 1 Web ACL + 2 Rules (short-term testing) | - $5.00/Web ACL/month (pro-rated)<br>- $1.00/Rule/month (pro-rated) | $1.00 - $2.00 (Enabled for demos only) |
| **AWS Backup & KMS** | - Backup for 2 GB DynamoDB table | - $0.10/GB backup storage | $0.20 |
| **Amazon GuardDuty** | - Lab scale monitoring | - 30-day Free Trial | $0.00 |
| **AWS X-Ray** | - 20,000 traces/month | - Free Tier (first 100,000 traces) | $0.00 |
| **Stripe Sandbox** | - Simulate Stripe payments | - Free of charge | $0.00 |
| **TOTAL** | | | **~ $3.37 - $5.12 USD / Month** (Highly economical) |

---

### 6. RISK ASSESSMENT & MITIGATION

Key technical risks and their associated mitigation strategies are analyzed below:

| Technical Risk | Impact | Mitigating Controls | Contingency Plan |
|---|---|---|---|
| **Database Connection Limits** under peak load | Medium | Use **Amazon DynamoDB** (serverless NoSQL database) which supports auto-scaling and scales connection pools transparently. | Alarm on high write capacity utilization to trigger manual throughput audits. |
| **Data Loss from Webhook Failures or Double Charges** | High | - Use unique **Stripe Idempotency-Keys**.<br>- Validate Stripe-Signature Headers on incoming Webhooks.<br>- Apply **DynamoDB Conditional Writes** on state transitions.<br>- Buffer through **SQS DLQ** on delivery failures. | Admin script to re-drive messages from DLQ to SQS queue after root cause resolution. |
| **Credential Leakage** | High | Never commit credential configurations (e.g., `.env`) to GitHub. Store keys in **AWS Secrets Manager**. | Configure Secrets Manager automatic credential rotation; revoke leaked keys instantly. |
| **Project Delay due to Architecture Complexity** | High | Frame a clear MVP focused on Product catalog, Checkout, and Lex Chatbot. Notification systems can be mocked if schedule slips. | Short daily sync meetings (10 min) to clear blockers. |

---

### 7. ACCEPTANCE CRITERIA

The platform is accepted upon meeting these technical criteria:

1. **Edge & Frontend:** Route 53 domain loads frontend over HTTPS. WAF rejects requests exceeding 100 reqs/min. Frontend auto-deploys via Amplify.
2. **Identity:** Users sign up/in, receive email confirmations, and obtain valid JWT tokens. Cognito Authorizer blocks access on unauthorized requests (401).
3. **Catalog & Orders:** Products display descriptions and images fetched from S3. Order requests queue in SQS asynchronously. Persistent order errors migrate to DLQ.
4. **Checkout & Events:** Payments succeed in Stripe Sandbox. Idempotent keys prevent double transactions. Webhook verifies signatures. EventBridge routes events to notification queues.
5. **Chatbot:** Interactive Lex UI functions on the frontend, resolving intents and returning database records.
6. **Ops:** CloudWatch logs system stats. X-Ray displays complete trace maps across API Gateway, Lambda, SQS, EventBridge, and DynamoDB.
7. **Process:** Infrastructure is completely defined using AWS CDK IaC. Pipelines build and test pull requests on Git merges.