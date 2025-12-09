---
title: "Quản lý Parameters"
date: 2025-12-08
weight: 5
chapter: false
pre: "<b>5.2.5. </b>"
---

## Quản lý Parameters cho Nhiều Môi trường

### Tổng quan

Parameters cho phép chúng ta triển khai cùng infrastructure templates tới các môi trường khác nhau (staging, production) với các cấu hình khác nhau.

![Parameter Files](/images/5-Workshop/5.2-Prerequisite/Parameter_Files.png)

### Cấu trúc Parameter Files

Chúng ta tổ chức parameters trong JSON files theo môi trường:

```
parameters/
├── staging.json
└── prod.json
```

### Ví dụ Parameter File

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

### Cơ chế Truyền Parameters

Parameters được chuyển đổi từ JSON sang CloudFormation format:

```bash
# Chuyển đổi JSON sang AWS CLI format
params_override=$(python3 -c "import json, sys; \
    data=json.load(open('$PARAMS_FILE')); \
    print(' '.join([f'ParameterKey={k},ParameterValue={v}' \
    for k,v in data.items()]))")

# Deploy với parameters
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name my-stack \
    --parameter-overrides $params_override
```


### Định nghĩa Parameters trong Template

```yaml
Parameters:
  Environment:
    Type: String
    Default: staging
    AllowedValues: [staging, prod]
  
  CorsOrigin:
    Type: String
    Default: "*"
  
  LogRetentionDays:
    Type: Number
    Default: 7
```

### Chiến lược Tách Môi trường

| Parameter | Staging | Production | Lý do |
|-----------|---------|------------|-------|
| **CorsOrigin** | `*` | `https://domain.com` | Bảo mật |
| **LogRetention** | 7 ngày | 30 ngày | Chi phí vs tuân thủ |
| **DebugLogs** | true | false | Hiệu suất |
| **MinCapacity** | 1 | 2 | Tính sẵn sàng |
| **MaxCapacity** | 10 | 100 | Scale |

### Xử lý Dữ liệu Nhạy cảm

**Cách hiện tại** (Cơ bản):
```json
{
  "DatabasePassword": "hardcoded-password"
}
```

**Cách đề xuất** (Bảo mật):

**1. AWS Systems Manager Parameter Store**:
```yaml
Parameters:
  DatabasePasswordSSM:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /travelguide/staging/db-password
```

**2. AWS Secrets Manager**:
```yaml
Resources:
  DatabaseSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      GenerateSecretString:
        PasswordLength: 32
```

### Best Practices

1. **Không bao giờ commit secrets** vào Git
2. **Validate parameters** trước deployment
3. **Document parameters** trong README
4. **Dùng parameter constraints**
5. **Environment-specific naming**

### Điểm chính

1. **Parameter files** cho phép cấu hình theo môi trường
2. **JSON format** dễ đọc và maintain
3. **Conversion script** chuyển JSON sang CloudFormation format
4. **Sensitive data** nên dùng Parameter Store hoặc Secrets Manager
5. **Validation** ngăn chặn deployment errors
