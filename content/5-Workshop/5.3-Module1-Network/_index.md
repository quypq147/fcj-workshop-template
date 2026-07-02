---
title: "Module 1: Core Network"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Module 1: Core Network (VPC & Subnets)

In this module, you will construct the networking foundation using a VPC distributed across 2 Availability Zones.

Create `lib/network-stack.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import { Construct } from 'constructs';

export class NetworkStack extends cdk.Stack {
  public readonly vpc: ec2.IVpc;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Initialize VPC on 2 Availability Zones for High Availability
    this.vpc = new ec2.Vpc(this, 'ECommerceVpc', {
      maxAzs: 2,
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/16'),
      subnetConfiguration: [
        {
          name: 'Public', // Public Subnet for NAT Gateways
          subnetType: ec2.SubnetType.PUBLIC,
          cidrMask: 24,
        },
        {
          name: 'Private', // Private Subnet for Lambdas and database endpoints
          subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
          cidrMask: 24,
        }
      ],
      natGateways: 1, // Single NAT Gateway for Dev to minimize cost
    });

    new cdk.CfnOutput(this, 'VpcIdOutput', {
      value: this.vpc.vpcId,
      description: 'VPC ID of the E-Commerce Platform',
    });
  }
}
```
