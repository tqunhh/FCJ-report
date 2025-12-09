---
title: "Deployment Orchestration"
date: 2025-12-08
weight: 4
chapter: false
pre: "<b>5.2.4. </b>"
---

## Deployment Orchestration với Bash Scripts

### Tổng quan

Deployment orchestration tự động hóa quá trình triển khai nhiều CloudFormation stacks theo đúng thứ tự với xử lý lỗi và validation phù hợp.

### Kiến trúc Deployment Scripts

```
scripts/
├── deploy.sh              # Script orchestration chính
├── deploy-core.sh         # Triển khai core stack
├── deploy-service.sh      # Template triển khai service stack
└── utils.sh               # Shared utility functions
```

### Script Orchestration Chính

**deploy.sh** - Điều phối toàn bộ deployment:

```bash
#!/bin/bash
set -e  # Thoát khi có lỗi
set -o pipefail  # Bắt lỗi trong pipes

# Cấu hình
ENVIRONMENT=${1:-staging}
REGION=${2:-us-east-1}
CORE_STACK_NAME="travel-guide-core-${ENVIRONMENT}"

echo "🚀 Bắt đầu deployment tới ${ENVIRONMENT}"

# Bước 1: Setup deployment bucket
echo "📦 Thiết lập deployment bucket..."
./scripts/setup-bucket.sh $ENVIRONMENT $REGION

# Bước 2: Deploy Core Stack
echo "🏗️  Deploying Core Stack..."
./scripts/deploy-core.sh $ENVIRONMENT $REGION

# Đợi core stack hoàn thành
echo "⏳ Đợi Core Stack..."
aws cloudformation wait stack-create-complete \
    --stack-name $CORE_STACK_NAME \
    --region $REGION

echo "✅ Core Stack deployed thành công"

# Bước 3: Deploy Service Stacks
SERVICES=("auth" "articles" "media" "ai" "gallery" "notification")

for service in "${SERVICES[@]}"; do
    echo "🔧 Deploying ${service} service..."
    ./scripts/deploy-service.sh $service $ENVIRONMENT $REGION
done

echo "✅ Tất cả services deployed thành công"
```


### Xử lý Lỗi

**Xử lý lỗi toàn diện**:

```bash
#!/bin/bash

# Thoát khi có lỗi
set -e

# Thoát khi biến undefined
set -u

# Bắt lỗi trong pipes
set -o pipefail

# Cleanup function
cleanup() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        echo "❌ Deployment thất bại với exit code: $exit_code"
        echo "📋 Kiểm tra CloudFormation events:"
        echo "aws cloudformation describe-stack-events --stack-name $STACK_NAME"
    fi
}

# Đăng ký cleanup khi exit
trap cleanup EXIT
```

### Chiến lược Rollback

**Automatic Rollback**:
```bash
# CloudFormation tự động rollback khi thất bại
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name my-stack \
    --disable-rollback false  # Hành vi mặc định
```

**Manual Rollback**:
```bash
# Xóa failed stack
aws cloudformation delete-stack --stack-name my-stack

# Redeploy phiên bản trước
git checkout v1.2.3
./deploy.sh staging
```

### Các bước Validation

**Pre-deployment validation**:

```bash
# 1. Validate template syntax
aws cloudformation validate-template \
    --template-body file://template.yaml

# 2. Lint template
cfn-lint template.yaml

# 3. Kiểm tra parameter file
jq empty < parameters/staging.json

# 4. Verify AWS credentials
aws sts get-caller-identity
```

### Parallel Deployment

**Deploy services song song** (nhanh hơn):

```bash
#!/bin/bash

SERVICES=("auth" "articles" "media" "ai" "gallery" "notification")

# Deploy tất cả services ở background
for service in "${SERVICES[@]}"; do
    ./scripts/deploy-service.sh $service $ENVIRONMENT $REGION &
done

# Đợi tất cả background jobs
wait

echo "✅ Tất cả services deployed"
```

### Điểm chính

1. **Orchestration scripts** tự động hóa multi-stack deployment
2. **Error handling** ngăn chặn partial deployments
3. **Validation** bắt lỗi trước khi deployment
4. **Rollback strategies** cho phép phục hồi nhanh
5. **Parallel deployment** tăng tốc quá trình
6. **Logging** giúp debug issues

### Best Practices

1. **Luôn validate** templates trước deployment
2. **Dùng set -e** để thoát khi có lỗi
3. **Implement retry logic** cho transient failures
4. **Log tất cả operations** để debugging
5. **Test scripts** trong staging trước production
6. **Document deployment process** trong README
