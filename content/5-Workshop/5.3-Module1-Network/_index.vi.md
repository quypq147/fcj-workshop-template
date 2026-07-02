---
title: "Module 1: Mạng Core"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Module 1: Core Network (VPC & Subnets)

Trong phần này, bạn sẽ thiết lập nền tảng mạng bảo mật sử dụng VPC trên 2 Availability Zones.

Tạo tệp `lib/network-stack.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import { Construct } from 'constructs';

export class NetworkStack extends cdk.Stack {
  public readonly vpc: ec2.IVpc;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Khởi tạo VPC trên 2 Availability Zones để đạt High Availability
    this.vpc = new ec2.Vpc(this, 'ECommerceVpc', {
      maxAzs: 2,
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/16'),
      subnetConfiguration: [
        {
          name: 'Public', // Subnet công cộng dành cho NAT Gateways
          subnetType: ec2.SubnetType.PUBLIC,
          cidrMask: 24,
        },
        {
          name: 'Private', // Subnet riêng tư cho các hàm Lambda và endpoint database
          subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
          cidrMask: 24,
        }
      ],
      natGateways: 1, // Sử dụng 1 NAT Gateway ở môi trường Dev để tiết kiệm chi phí
    });

    new cdk.CfnOutput(this, 'VpcIdOutput', {
      value: this.vpc.vpcId,
      description: 'VPC ID of the E-Commerce Platform',
    });
  }
}
```
