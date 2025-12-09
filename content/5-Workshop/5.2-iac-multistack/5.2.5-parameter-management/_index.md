---
title: "Parameter Management"
date: 2025-12-08
weight: 5
chapter: false
pre: "<b>5.2.5. </b>"
---

## Parameter Management for Multiple Environments

### Overview

Parameters allow us to deploy the same infrastructure templates to different environments (staging, production) with different configurations.

![Parameter Files](/images/5-Workshop/5.2-Prerequisite/Parameter_Files.png)

### Parameter Files Structure

We organize parameters in JSON files per environment:

```
parameters/
├── staging.json
└── prod.json
```

### Example Parameter File

**staging.json**:
```json
{
  "Environment": "staging",
  "CorsOrigin": "*",
  "AdminEmail": "admin-staging@example.com",
  "LogRetentionDays": "7",
  "EnableDebugLogs": "true",
  "MinCapacity": "1",
  "MaxCapacity": "10"
}
```

**prod.json**:
```json
{
  "Environment": "prod",
  "CorsOrigin": "https://travelguide.com",
  "AdminEmail": "admin@example.com",
  "LogRetentionDays": "30",
  "EnableDebugLogs": "false",
  "MinCapacity": "2",
  "MaxCapacity": "100"
}
```

### Parameter Passing Mechanism

Parameters are converted from JSON to CloudFormation format:

```bash
# Convert JSON to AWS CLI format
params_override=$(python -c "import json, sys; \
    data=json.load(sys.stdin); \
    print(' '.join([f'ParameterKey={k},ParameterValue={v}' \
    for k,v in data.items()]))" < $PARAMS_FILE)

# Deploy with parameters
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name my-stack \
    --parameter-overrides $params_override
```

### Template Parameter Definitions

In CloudFormation template:

```yaml
Parameters:
  Environment:
    Type: String
    Default: staging
    AllowedValues: [staging, prod]
    Description: Deployment environment
  
  CorsOrigin:
    Type: String
    Default: "*"
    Description: CORS origin for API Gateway
  
  AdminEmail:
    Type: String
    Description: Admin email for notifications
  
  LogRetentionDays:
    Type: Number
    Default: 7
    Description: CloudWatch Logs retention in days
  
  EnableDebugLogs:
    Type: String
    Default: "false"
    AllowedValues: ["true", "false"]
    Description: Enable debug logging
```

### Using Parameters in Resources

```yaml
Resources:
  CreateArticleFunction:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          ENVIRONMENT: !Ref Environment
          DEBUG: !Ref EnableDebugLogs
          CORS_ORIGIN: !Ref CorsOrigin
      
  ApiGateway:
    Type: AWS::Serverless::Api
    Properties:
      Cors:
        AllowOrigin: !Sub "'${CorsOrigin}'"
        AllowHeaders: "'*'"
  
  LogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      RetentionInDays: !Ref LogRetentionDays
```

### Environment Separation Strategy

| Parameter | Staging | Production | Reason |
|-----------|---------|------------|--------|
| **CorsOrigin** | `*` | `https://domain.com` | Security |
| **LogRetention** | 7 days | 30 days | Cost vs compliance |
| **DebugLogs** | true | false | Performance |
| **MinCapacity** | 1 | 2 | Availability |
| **MaxCapacity** | 10 | 100 | Scale |

### Sensitive Data Handling

**Current Approach** (Basic):
```json
{
  "DatabasePassword": "hardcoded-password"
}
```

**Recommended Approach** (Secure):

**1. AWS Systems Manager Parameter Store**:
```yaml
Parameters:
  DatabasePasswordSSM:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /travelguide/staging/db-password

Resources:
  Function:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          DB_PASSWORD: !Ref DatabasePasswordSSM
```

**2. AWS Secrets Manager**:
```yaml
Resources:
  DatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Name: !Sub '${AWS::StackName}-db-secret'
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: "password"
        PasswordLength: 32
  
  Function:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          SECRET_ARN: !Ref DatabaseSecret
```

### Deployment Script with Parameters

```bash
#!/bin/bash
set -e

ENVIRONMENT=$1  # staging or prod
PARAMS_FILE="parameters/${ENVIRONMENT}.json"

if [ ! -f "$PARAMS_FILE" ]; then
    echo "Error: Parameter file not found: $PARAMS_FILE"
    exit 1
fi

# Convert JSON to parameter overrides
params_override=$(python -c "import json, sys; \
    data=json.load(sys.stdin); \
    print(' '.join([f'ParameterKey={k},ParameterValue={v}' \
    for k,v in data.items()]))" < $PARAMS_FILE)

# Deploy stack
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name "travel-guide-${ENVIRONMENT}" \
    --parameter-overrides $params_override \
    --capabilities CAPABILITY_IAM \
    --no-fail-on-empty-changeset

echo "✅ Deployed to ${ENVIRONMENT}"
```

### Best Practices

1. **Never commit secrets** to Git
   - Use `.gitignore` for sensitive parameter files
   - Use Parameter Store or Secrets Manager

2. **Validate parameters** before deployment
   ```bash
   # Validate JSON syntax
   jq empty < parameters/staging.json
   ```

3. **Document parameters** in README
   ```markdown
   ## Parameters
   - `Environment`: Deployment environment (staging/prod)
   - `CorsOrigin`: CORS origin for API Gateway
   - `AdminEmail`: Email for admin notifications
   ```

4. **Use parameter constraints**
   ```yaml
   Parameters:
     InstanceType:
       Type: String
       AllowedValues: [t3.micro, t3.small, t3.medium]
       Default: t3.micro
   ```

5. **Environment-specific naming**
   ```yaml
   Resources:
     Bucket:
       Type: AWS::S3::Bucket
       Properties:
         BucketName: !Sub 'travelguide-${Environment}-images'
   ```

### Key Takeaways

1. **Parameter files** enable environment-specific configurations
2. **JSON format** is easy to read and maintain
3. **Conversion script** transforms JSON to CloudFormation format
4. **Sensitive data** should use Parameter Store or Secrets Manager
5. **Validation** prevents deployment errors
6. **Documentation** helps team understand parameters

### Common Pitfalls

❌ **Hardcoding values** in templates
```yaml
# Bad
Environment:
  Variables:
    API_URL: "https://api.staging.example.com"
```

✅ **Use parameters**
```yaml
# Good
Environment:
  Variables:
    API_URL: !Sub "https://api.${Environment}.example.com"
```

❌ **Committing secrets** to Git
```json
{
  "DatabasePassword": "super-secret-password"
}
```

✅ **Use Secrets Manager**
```yaml
DatabaseSecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    GenerateSecretString: {}
```
