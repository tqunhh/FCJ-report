---
title: "Cross-Stack References"
date: 2025-12-08
weight: 3
chapter: false
pre: "<b>5.2.3. </b>"
---

## Cross-Stack References Implementation

### Overview

Cross-stack references allow one CloudFormation stack to use resources created by another stack. This is essential for the multi-stack pattern where service stacks need to reference resources from the core stack.

### CloudFormation Exports Mechanism

The **exporting stack** (Core Stack) exposes resources using `Outputs` with `Export`:

```yaml
# Core Stack - template.yaml
Outputs:
  ArticlesTableName:
    Description: Articles DynamoDB Table Name
    Value: !Ref ArticlesTable
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableName'
  
  ArticlesTableArn:
    Description: Articles DynamoDB Table ARN
    Value: !GetAtt ArticlesTable.Arn
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableArn'
  
  ImagesBucketName:
    Description: S3 Bucket for Images
    Value: !Ref ImagesBucket
    Export:
      Name: !Sub '${AWS::StackName}-ImagesBucketName'
  
  UserPoolId:
    Description: Cognito User Pool ID
    Value: !Ref UserPool
    Export:
      Name: !Sub '${AWS::StackName}-UserPoolId'
```

### CloudFormation Imports Mechanism

The **importing stack** (Service Stack) references exported values using `!ImportValue`:

```yaml
# Article Service Stack - template.yaml
Parameters:
  CoreStackName:
    Type: String
    Description: Name of the core stack to import values from
    Default: travel-guide-core-staging

Resources:
  CreateArticleFunction:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          ARTICLES_TABLE: !ImportValue
            'Fn::Sub': '${CoreStackName}-ArticlesTableName'
          IMAGES_BUCKET: !ImportValue
            'Fn::Sub': '${CoreStackName}-ImagesBucketName'
          USER_POOL_ID: !ImportValue
            'Fn::Sub': '${CoreStackName}-UserPoolId'
      
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !ImportValue
              'Fn::Sub': '${CoreStackName}-ArticlesTableName'
        - S3CrudPolicy:
            BucketName: !ImportValue
              'Fn::Sub': '${CoreStackName}-ImagesBucketName'
```

### Export Naming Conventions

**Best Practice**: Include stack name in export name to avoid conflicts

❌ **Bad** (can conflict across environments):
```yaml
Export:
  Name: ArticlesTableName  # Conflicts if multiple environments
```

✅ **Good** (unique per environment):
```yaml
Export:
  Name: !Sub '${AWS::StackName}-ArticlesTableName'
  # Results in: travel-guide-core-staging-ArticlesTableName
```

**Naming Pattern**:
```
{StackName}-{ResourceType}{ResourceName}

Examples:
- travel-guide-core-staging-ArticlesTableName
- travel-guide-core-staging-ArticlesTableArn
- travel-guide-core-prod-ImagesBucketName
```

### Complete Example

#### Core Stack Exports

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Travel Guide - Core Infrastructure

Resources:
  ArticlesTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub '${AWS::StackName}-articles'
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: articleId
          AttributeType: S
      KeySchema:
        - AttributeName: articleId
          KeyType: HASH
  
  ImagesBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${AWS::StackName}-images'
      CorsConfiguration:
        CorsRules:
          - AllowedOrigins: ['*']
            AllowedMethods: [GET, PUT, POST]
            AllowedHeaders: ['*']

Outputs:
  # Table Name Export
  ArticlesTableName:
    Description: Articles Table Name
    Value: !Ref ArticlesTable
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableName'
  
  # Table ARN Export
  ArticlesTableArn:
    Description: Articles Table ARN
    Value: !GetAtt ArticlesTable.Arn
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableArn'
  
  # Bucket Name Export
  ImagesBucketName:
    Description: Images Bucket Name
    Value: !Ref ImagesBucket
    Export:
      Name: !Sub '${AWS::StackName}-ImagesBucketName'
  
  # Bucket ARN Export
  ImagesBucketArn:
    Description: Images Bucket ARN
    Value: !GetAtt ImagesBucket.Arn
    Export:
      Name: !Sub '${AWS::StackName}-ImagesBucketArn'
```

#### Service Stack Imports

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Travel Guide - Article Service

Parameters:
  CoreStackName:
    Type: String
    Description: Core stack name
    Default: travel-guide-core-staging

Resources:
  # Lambda Function using imported values
  CreateArticleFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./src
      Handler: create_article.handler
      Runtime: python3.11
      Environment:
        Variables:
          # Import table name
          ARTICLES_TABLE: !ImportValue
            'Fn::Sub': '${CoreStackName}-ArticlesTableName'
          # Import bucket name
          IMAGES_BUCKET: !ImportValue
            'Fn::Sub': '${CoreStackName}-ImagesBucketName'
      
      Policies:
        # Grant DynamoDB access using imported ARN
        - Statement:
            - Effect: Allow
              Action:
                - dynamodb:PutItem
                - dynamodb:GetItem
                - dynamodb:UpdateItem
              Resource: !ImportValue
                'Fn::Sub': '${CoreStackName}-ArticlesTableArn'
        
        # Grant S3 access using imported ARN
        - Statement:
            - Effect: Allow
              Action:
                - s3:PutObject
                - s3:GetObject
              Resource:
                - !Sub
                  - '${BucketArn}/*'
                  - BucketArn: !ImportValue
                      'Fn::Sub': '${CoreStackName}-ImagesBucketArn'
```

### Limitations and Workarounds

#### Limitation 1: Cannot Delete Exporting Stack

**Problem**: Cannot delete core stack while service stacks are importing its values.

**Error**:
```
Export travel-guide-core-staging-ArticlesTableName cannot be deleted 
as it is in use by travel-guide-article-service-staging
```

**Workaround**:
1. Delete all importing stacks first
2. Then delete the exporting stack

```bash
# Delete service stacks first
aws cloudformation delete-stack --stack-name travel-guide-article-service-staging
aws cloudformation delete-stack --stack-name travel-guide-media-service-staging

# Wait for deletion
aws cloudformation wait stack-delete-complete --stack-name travel-guide-article-service-staging

# Now delete core stack
aws cloudformation delete-stack --stack-name travel-guide-core-staging
```

#### Limitation 2: Cannot Change Export Name

**Problem**: Cannot rename or delete an export while it's being imported.

**Workaround**:
1. Create new export with new name
2. Update all importing stacks to use new export
3. Delete old export

```yaml
# Step 1: Add new export (keep old one)
Outputs:
  ArticlesTableName:  # Old export (keep for now)
    Value: !Ref ArticlesTable
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableName'
  
  ArticlesTableNameV2:  # New export
    Value: !Ref ArticlesTable
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTable-Name'

# Step 2: Update service stacks to use new export
# Step 3: Remove old export from core stack
```

#### Limitation 3: Cross-Region Not Supported

**Problem**: Cannot import values from stacks in different regions.

**Workaround**: Use SSM Parameter Store for cross-region sharing:

```yaml
# Core Stack (us-east-1)
Resources:
  TableNameParameter:
    Type: AWS::SSM::Parameter
    Properties:
      Name: /travelguide/core/articles-table-name
      Type: String
      Value: !Ref ArticlesTable

# Service Stack (us-west-2)
Parameters:
  ArticlesTableName:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /travelguide/core/articles-table-name
```

### Best Practices

1. **Always include stack name** in export names
   ```yaml
   Export:
     Name: !Sub '${AWS::StackName}-ResourceName'
   ```

2. **Export both name and ARN** for resources
   ```yaml
   Outputs:
     TableName:
       Value: !Ref Table
       Export:
         Name: !Sub '${AWS::StackName}-TableName'
     TableArn:
       Value: !GetAtt Table.Arn
       Export:
         Name: !Sub '${AWS::StackName}-TableArn'
   ```

3. **Document exports** in README
   ```markdown
   ## Core Stack Exports
   - `{StackName}-ArticlesTableName`: DynamoDB table name
   - `{StackName}-ArticlesTableArn`: DynamoDB table ARN
   - `{StackName}-ImagesBucketName`: S3 bucket name
   ```

4. **Use parameters** for core stack name
   ```yaml
   Parameters:
     CoreStackName:
       Type: String
       Default: travel-guide-core-staging
   ```

5. **Plan exports carefully** - they're hard to change later

### Viewing Exports

**AWS Console**:
- CloudFormation → Stacks → Select Stack → Outputs tab

**AWS CLI**:
```bash
# List all exports
aws cloudformation list-exports

# List exports from specific stack
aws cloudformation describe-stacks \
    --stack-name travel-guide-core-staging \
    --query 'Stacks[0].Outputs'
```

### Key Takeaways

1. **Exports** allow sharing resources between stacks
2. **Imports** reference exported values using `!ImportValue`
3. **Naming convention** prevents conflicts: `{StackName}-{ResourceName}`
4. **Limitations** exist: can't delete exporting stack, can't change export names
5. **Workarounds** available for most limitations
6. **Documentation** is crucial for team understanding
