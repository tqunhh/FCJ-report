---
title: "Cross-Stack References"
date: 2025-12-08
weight: 3
chapter: false
pre: "<b>5.2.3. </b>"
---

## Triển khai Cross-Stack References

### Tổng quan

Cross-stack references cho phép một CloudFormation stack sử dụng resources được tạo bởi stack khác. Điều này rất quan trọng cho multi-stack pattern khi service stacks cần tham chiếu resources từ core stack.

### Cơ chế CloudFormation Exports

**Stack xuất** (Core Stack) expose resources bằng `Outputs` với `Export`:

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
```

### Cơ chế CloudFormation Imports

**Stack nhập** (Service Stack) tham chiếu exported values bằng `!ImportValue`:

```yaml
# Article Service Stack
Parameters:
  CoreStackName:
    Type: String
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
```


### Quy ước Đặt tên Export

**Best Practice**: Bao gồm tên stack trong export name để tránh xung đột

❌ **Không tốt** (có thể xung đột giữa các môi trường):
```yaml
Export:
  Name: ArticlesTableName
```

✅ **Tốt** (unique cho mỗi môi trường):
```yaml
Export:
  Name: !Sub '${AWS::StackName}-ArticlesTableName'
  # Kết quả: travel-guide-core-staging-ArticlesTableName
```

### Giới hạn và Cách khắc phục

#### Giới hạn 1: Không thể xóa Stack đang Export

**Vấn đề**: Không thể xóa core stack khi service stacks đang import values của nó.

**Cách khắc phục**:
```bash
# Xóa service stacks trước
aws cloudformation delete-stack --stack-name travel-guide-article-service-staging

# Đợi xóa hoàn tất
aws cloudformation wait stack-delete-complete --stack-name travel-guide-article-service-staging

# Sau đó xóa core stack
aws cloudformation delete-stack --stack-name travel-guide-core-staging
```

#### Giới hạn 2: Không thể thay đổi tên Export

**Vấn đề**: Không thể đổi tên hoặc xóa export khi đang được import.

**Cách khắc phục**: Tạo export mới, update importing stacks, sau đó xóa export cũ.

#### Giới hạn 3: Không hỗ trợ Cross-Region

**Vấn đề**: Không thể import values từ stacks ở regions khác.

**Cách khắc phục**: Dùng SSM Parameter Store cho cross-region sharing.

### Best Practices

1. **Luôn bao gồm stack name** trong export names
2. **Export cả name và ARN** cho resources
3. **Document exports** trong README
4. **Dùng parameters** cho core stack name
5. **Plan exports cẩn thận** - khó thay đổi sau này

### Điểm chính

1. **Exports** cho phép chia sẻ resources giữa stacks
2. **Imports** tham chiếu exported values bằng `!ImportValue`
3. **Naming convention** tránh xung đột: `{StackName}-{ResourceName}`
4. **Giới hạn** tồn tại nhưng có cách khắc phục
5. **Documentation** rất quan trọng
