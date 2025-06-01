---
title: "Building at Scale: Multi-Account AWS Infrastructure with CloudFormation StackSets"
seoTitle: "Scale AWS Infrastructure Across Multiple Accounts"
seoDescription: "Learn how to deploy consistent infrastructure across multiple AWS accounts using CloudFormation StackSets. A step-by-step guide for enterprise environments."
datePublished: Thu May 08 2025 20:55:34 GMT+0000 (Coordinated Universal Time)
cuid: cmafujpiz000309jy0b7pguds
slug: building-at-scale-multi-account-aws-infrastructure-with-cloudformation-stacksets
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1746732221624/40ddd0ee-b3a3-466c-b420-425b62dc4a93.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1746737633159/9ca982dc-a89c-44c9-bdd6-b822fcf5ae85.png
tags: cloudformation, aws, cloudformation-stacks, stacksets

---

In today's complex enterprise environments, managing infrastructure across multiple AWS accounts has become standard practice. Whether you're implementing a landing zone architecture, enforcing organizational policies, or deploying consistent resources, doing this manually across dozens or hundreds of accounts quickly becomes unmanageable. This is where AWS CloudFormation StackSets shines—allowing you to deploy identical stacks across multiple accounts and regions with a single operation.

In this comprehensive guide, we'll walk through implementing cross-account infrastructure using CloudFormation StackSets. You'll learn the practical steps to establish a foundation for multi-account resource management that can scale with your organization.

---

## Why Multi-Account Architecture Matters

Before diving into implementation, let's understand why organizations adopt multi-account strategies:

* **Security Isolation**: Separate workloads and data based on security requirements
    
* **Cost Management**: Track and allocate costs to specific business units
    
* **Blast Radius Containment**: Limit the impact of incidents to a single account
    
* **Governance Alignment**: Match AWS accounts to organizational structure
    
* **Compliance Requirements**: Segregate regulated workloads from non-regulated ones
    

However, this architecture introduces a significant challenge: how do you efficiently deploy and manage resources across all these accounts? Enter `CloudFormation StackSets`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746733465563/6e5275d2-23e5-45d6-87c7-c237c55520d4.png align="center")

---

## Understanding Self-Managed StackSets

For cross-account deployments, AWS offers two permission models: `SELF_MANAGED` and `SERVICE_MANAGED`. In this implementation, we'll use the `SELF_MANAGED` approach, which requires creating two IAM roles:

* `AdministrationRole`: Lives in the management account(root account of the organization) and allows CloudFormation to assume execution roles in target accounts
    
* `ExecutionRole`: Lives in each target account and grants permissions to create resources
    

Let's break down the implementation step by step.

### Step 1: Creating the Administration Role

First, we need to create the CloudFormation StackSet Administration Role in the management account:

1. Navigate to `CloudFormation` &gt; `StackSets` &gt; `Create StackSet`
    
2. Leave the `IAM` role and `ExecutionRole` fields empty (we're creating these roles)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746733595835/a8ff3152-c546-41c8-b42f-91613f9720fb.png align="center")
    
3. Upload the following template:
    
    ```yaml
    AWSTemplateFormatVersion: '2010-09-09'
    Description: >
      Creates the CloudFormation StackSet Administration Role in the management account.
      This role allows CloudFormation to assume execution roles in target accounts.
    Parameters:
      RootAccountId:
        Type: String
        Description: The root account ID of the main AWS account
    Resources:
      StackSetAdministrationRole:
        Type: AWS::IAM::Role
        Properties:
          RoleName: !Sub "AWSCloudFormationStackSetAdministrationRole-${RootAccountId}"
          AssumeRolePolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Principal:
                  Service: "cloudformation.amazonaws.com"
                Action: "sts:AssumeRole"
          Policies:
            - PolicyName: "AssumeExecutionRolePolicy"
              PolicyDocument:
                Version: "2012-10-17"
                Statement:
                  - Effect: Allow
                    Action: "sts:AssumeRole"
                    Resource: "arn:aws:iam::*:role/AWSCloudFormationStackSetExecutionRole-${RootAccountId}"
    Outputs:
      AdministrationRoleArn:
        Description: "ARN of the StackSet Administration Role"
        Value: !GetAtt StackSetAdministrationRole.Arn
    ```
    
4. Complete the StackSet creation by specifying a name, description, and your management account ID under "Account numbers"
    
5. Specify the deployment region and submit.
    

This template creates an IAM role that CloudFormation service can assume to orchestrate cross-account deployments. The policy attached to this role allows it to assume the execution role in target accounts.

## Step 2: Creating the Execution Role

Next, we need to create the CloudFormation StackSet Execution Role that will be deployed to all target accounts:

1. Navigate to `CloudFormation` &gt; `StackSets` &gt; `Create StackSet`
    
2. Leave the `IAM` role and `ExecutionRole` fields empty (same as while creating Administrative Role)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746733797055/91f72fda-f940-406b-a293-be29d57b318b.png align="center")
    
3. Upload a template that creates the execution role with permissions for your specific resource types:
    
    ```yaml
    AWSTemplateFormatVersion: '2010-09-09'
    Description: >
      Creates the CloudFormation StackSet Execution Role in target accounts.
      This role is assumed by the administration role and has permissions for
      resource management.
    Parameters:
      RootAccountId:
        Type: String
        Description: The root account ID of the main AWS account
    Resources:
      StackSetExecutionRole:
        Type: AWS::IAM::Role
        Properties:
          RoleName: !Sub "AWSCloudFormationStackSetExecutionRole-${RootAccountId}"
          AssumeRolePolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Principal:
                  AWS:
                    - !Sub "arn:aws:iam::${RootAccountId}:root"
                Action: "sts:AssumeRole"
          ManagedPolicyArns:
            - "arn:aws:iam::aws:policy/AdministratorAccess"  # Be as restrictive as possible for production env
    Outputs:
      ExecutionRoleArn:
        Description: "ARN of the StackSet Execution Role"
        Value: !GetAtt StackSetExecutionRole.Arn
    ```
    
4. Complete the `StackSet` creation with a name and description
    
5. Under "Account numbers," add all target accounts as comma-separated values
    
6. Specify the deployment region and submit.
    

### Step 3: Deploying Infrastructure Across Accounts

Now that we have the foundation set up, we can deploy actual infrastructure across our accounts. Let's create a simple example that establishes a standardized S3 bucket for logging in each account:

1. Navigate to `CloudFormation` &gt; `StackSets` &gt; `Create StackSet`
    
2. Select the `AdministrationRole` and specify the `ExecutionRole` name
    
3. Upload a template that creates your resources:
    
    ```yaml
    AWSTemplateFormatVersion: '2010-09-09'
    Description: Creates standardized resources across accounts
    Resources:
      CentralizedLogBucket:
        Type: AWS::S3::Bucket
        Properties:
          BucketName: !Sub "centralized-logs-${AWS::AccountId}"
          VersioningConfiguration:
            Status: Enabled
          BucketEncryption:
            ServerSideEncryptionConfiguration:
              - ServerSideEncryptionByDefault:
                  SSEAlgorithm: AES256
          PublicAccessBlockConfiguration:
            BlockPublicAcls: true
            BlockPublicPolicy: true
            IgnorePublicAcls: true
            RestrictPublicBuckets: true
          LifecycleConfiguration:
            Rules:
              - Id: TransitionToGlacierAndEventuallyExpire
                Status: Enabled
                ExpirationInDays: 2555  # ~7 years
                Transitions:
                  - TransitionInDays: 90
                    StorageClass: STANDARD_IA
                  - TransitionInDays: 365
                    StorageClass: GLACIER
      LogBucketPolicy:
        Type: AWS::S3::BucketPolicy
        Properties:
          Bucket: !Ref CentralizedLogBucket
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Sid: DenyUnencryptedObjectUploads
                Effect: Deny
                Principal: '*'
                Action: 's3:PutObject'
                Resource: !Sub '${CentralizedLogBucket.Arn}/*'
                Condition:
                  StringNotEquals:
                    's3:x-amz-server-side-encryption': 'AES256'
    Outputs:
      LogBucketName:
        Description: "Centralized logging bucket name"
        Value: !Ref CentralizedLogBucket
      LogBucketArn:
        Description: "Centralized logging bucket ARN"
        Value: !GetAtt CentralizedLogBucket.Arn
    ```
    
4. Add your target accounts and deploy to the desired regions.
    

### Real-World Applications

The simple S3 bucket example is just the beginning. Here are some common infrastructure patterns deployed with StackSets:

#### 1\. Network Infrastructure

Deploy consistent VPC configurations across accounts:

```yaml
Resources:
  StandardVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Sub "10.${AccountNumberSuffix}.0.0/16"
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub "standard-vpc-${AWS::AccountId}"
  
  # Subnets, route tables, etc.
```

#### 2\. IAM Policies and Roles

Establish baseline permissions and security guardrails:

```yaml
Resources:
  ReadOnlyRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: "OrganizationReadOnlyRole"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              AWS: !Sub "arn:aws:iam::${ManagementAccountId}:root"
            Action: "sts:AssumeRole"
      ManagedPolicyArns:
        - "arn:aws:iam::aws:policy/ReadOnlyAccess"
```

---

## Architecture Design Patterns

When designing multi-account infrastructure, consider these patterns:

### Hub and Spoke

The management account (hub) deploys and manages resources in target accounts (spokes). This centralized approach works well for governance, security, and compliance resources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1746737598388/b28569a3-c1cb-4767-a20f-247e7fbf53d5.png align="center")

### Service-Specific Accounts

Dedicate accounts to specific shared services like networking, security, or logging. Use `StackSets` to establish connections between these service accounts and workload accounts.

### Regional Deployment Strategy

For global organizations, consider regional deployment strategies:

* Deploy to all regions simultaneously
    
* Deploy region by region
    
* Maintain region-specific configurations via parameters
    

---

## Advanced StackSets Features

### Stack Set Operations Options

CloudFormation StackSets provides several options to control deployment behavior:

* **Maximum Concurrent Accounts**: Control how many accounts receive updates simultaneously
    
* **Failure Tolerance**: Define how many account failures are acceptable before halting deployment
    
* **Region Concurrency**: Specify if regions should be updated in parallel or sequentially
    
* **Retain Stacks**: Choose if stacks should remain when removed from the StackSet
    

### Parameter Overrides

Use parameter overrides to customize deployments for specific accounts or regions:

```bash
aws cloudformation create-stack-set \
  --stack-set-name NetworkInfrastructure \
  --template-body file://network-template.yaml \
  --parameters \
    ParameterKey=VpcCidr,ParameterValue=10.0.0.0/16 \
    ParameterKey=Environment,ParameterValue=Production
```

Then override parameters for specific deployments:

```bash
aws cloudformation create-stack-instances \
  --stack-set-name NetworkInfrastructure \
  --accounts 111122223333 \
  --regions us-east-1 \
  --parameter-overrides \
    ParameterKey=VpcCidr,ParameterValue=10.1.0.0/16 \
    ParameterKey=Environment,ParameterValue=Development
```

### Integration with AWS Organizations

If using AWS Organizations, consider moving to SERVICE\_MANAGED permissions mode:

```bash
aws cloudformation create-stack-set \
  --stack-set-name OrganizationDeployment \
  --template-body file://template.yaml \
  --permission-model SERVICE_MANAGED \
  --auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false
```

This allows you to deploy to organizations or organizational units (OUs) rather than specifying account IDs:

```bash
aws cloudformation create-stack-instances \
  --stack-set-name OrganizationDeployment \
  --deployment-targets OrganizationalUnitIds=ou-examplerootid-exampleouid \
  --regions us-east-1
```

---

# Conclusion and Key Takeaways

AWS CloudFormation StackSets provides a powerful foundation for managing resources at scale across your multi-account AWS environment. By implementing the patterns and practices outlined in this guide, you can:

* Enforce consistent configurations across your organization
    
* Reduce management overhead through automation
    
* Maintain security and compliance at scale
    
* Accelerate new account onboarding and provisioning
    

As your organization grows, this infrastructure-as-code approach to multi-account management becomes increasingly valuable, enabling your team to focus on innovation rather than repetitive deployment tasks.

Whether you're managing a handful of accounts or hundreds, CloudFormation StackSets offers the scalability, flexibility, and control needed for enterprise-grade AWS environments.

So this was it for this blog everyone, I hope you’re able to understand more about AWS Cloudformation stacksets and how we can create infrastructure in multiple AWS accounts.

If you liked this blog, a like would be appreciated, and do let me know areas of improvement. Also, if you want me to blog about some other services of **AWS** or **DevOps**, do let me know in the comments or you can reach me on Twitter.