---
title: "Event 2"
date: "2025-01-01"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Bài thu hoạch “DevOps on AWS Workshop”

### Mục Đích Của Sự Kiện

- Trang bị kiến thức DevOps end-to-end trên AWS
- Xây dựng CI/CD pipeline hoàn chỉnh
- Làm quen và thực hành Infrastructure as Code (IaC)
- Triển khai và vận hành container-based applications
- Hiểu đúng về monitoring & observability
- Chia sẻ best practices, case studies và career path DevOps
### Danh Sách Diễn Giả

- **Bao Huynh** - AWS Community Builder
- **Thinh Nguyen** -AWS Community Builder
- **Vi Tran** - AWS Community Builder
### Nội Dung Nổi Bật

#### CI/CD Pipeline với AWS DevOps Services

Workshop đi từ nền tảng đến triển khai thực tế:

* Source Control
  * AWS CodeCommit
  * So sánh GitFlow và Trunk-based development
→ Hiểu rõ khi nào nên dùng mỗi strategy cho team size và release model khác nhau.

* Build & Test
  * Cấu hình AWS CodeBuild
  * Tích hợp unit test, integration test vào pipeline

* Deployment
  * Blue/Green deployment – zero downtime
  * Canary deployment – rollout có kiểm soát
  * Rolling updates – phù hợp cho ứng dụng stateless

 **Demo** CI/CD pipeline hoàn chỉnh là highlight lớn nhất, giúp thấy rõ toàn bộ luồng từ code commit đến production deployment, kèm theo rollback và error handling strategies.

#### Infrastructure as Code (IaC)
Phần IaC giúp thay đổi tư duy từ “click ops” sang “infrastructure as software”.

* AWS CloudFormation
  * Templates, stack management
  * Drift detection – cực kỳ quan trọng trong production
  * Nested stacks cho kiến trúc phức tạp
  
* AWS CDK
  *  Viết infrastructure bằng TypeScript/Python thay vì YAML/JSON
  *  Type safety, IDE support
  *  Reusable constructs và higher-level abstractions
  *  Testing infrastructure code
  
#### Container Services on AWS

Nội dung container được trình bày rất toàn diện:
* Docker Fundamentals
  * Containerization concepts
  * Microservices architecture
  * Best practices cho Dockerfile
  * Multi-stage builds
* Amazon ECR
  * Private container registry
  * Image security scanning
  * Lifecycle policies quản lý images
* Amazon ECS vs EKS
  * ECS: AWS-native, setup đơn giản, tích hợp sâu với AWS
  * EKS: Kubernetes standard, portable, learning curve cao hơn
  * Auto-scaling và deployment strategies cho từng service

#### Monitoring & Observability

* Một trong những phần thực tế và áp dụng ngay được:
  * Amazon CloudWatch
  * Metrics tiêu chuẩn & custom metrics
  * Log aggregation
  * Alarm với SNS notifications
  * Dashboard design cho từng stakeholder
* AWS X-Ray
  * Distributed tracing cho microservices
  * Service map visualization
  * Performance bottleneck identification
* Observability Best Practices
  * Three pillars: Metrics – Logs – Traces
  * Alerting strategies để tránh alert fatigue
  * On-call rotation & incident response
  
#### DevOps Best Practices & Case Studies

* Progressive Delivery
  * Feature flags
  * A/B testing
  * Canary analysis
* Automated Testing
  * Test pyramid: Unit → Integration → E2E
  * CI/CD integration
  * Quality gates
* Incident Management
  * Incident response procedures
  * Blameless postmortems
  * Learning from failures

### Những Gì Học Được

* Sau workshop, mình hiểu rõ hơn:
  * Cách thiết kế CI/CD pipeline production-ready trên AWS
  * Khi nào nên dùng Blue/Green, Canary hay Rolling deployments
  * Tư duy IaC và lựa chọn đúng tool cho từng giai đoạn
  * Chiến lược triển khai container phù hợp với workload
  * Tầm quan trọng của observability trong hệ thống phân tán
  * DevOps không chỉ là tooling mà là mindset: automation, collaboration, continuous improvement

#### Trải nghiệm cá nhân
**Ấn tượng chung**

Workshop mang tính intensive và rất practical. So với AI/ML workshop trước đó, DevOps workshop tập trung nhiều hơn vào vận hành thực tế và kỹ năng áp dụng ngay.

**Điểm nhấn**
  * CI/CD pipeline demo với Blue/Green deployment
  * AWS CDK – thật sự là “game changer”
  * Phần monitoring & observability cực kỳ sát thực tế
  * Case studies rất relatable

**Networking & Q&A**
  * Thảo luận về career paths trong DevOps
  * AWS certification roadmap (DevOps Engineer – Professional)
  * Chia sẻ kinh nghiệm từ các practitioners đang làm DevOps thực tế